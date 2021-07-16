---
title: Redis数据结构内部实现-字典
date: 2019-12-03 22:04:28
tags: [redis, 数据结构]
categories:
- 笔记
- Redis
---

# 字典

字典又称符号表，映射，是一种用于保存键值对的抽象数据结构。在字典中，一个键（key）可以和一个值（value）进行关联（或者说将键映射为值），这些关联的键和值就成为键值对。
字典中的每个键都是独一无二的，程序可以在字典中根据键查找与键关联的值，或者通过键来更新关联的值，又或者根据键来删除整个键值对。

在很多编程语言都会内置字典这个数据结构，由于Redis使用C实现的，C本身没有内置字典这种结构，因此Redis构建了自己的字典实现。

## 字典的实现
Redis的字典使用哈希表作为底层实现，一个哈希表里可以有多个哈希节点，而每个哈希节点保存一个键值对。

### 哈希表

```cpp
typedef struct dictht {
    //哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小掩码，用于计算索引值,总是等于size-1
    unsigned long sizemask;
    //该哈希表已有节点的数量
    unsigned long used;
} dictht;

```

![空的哈希表](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/empty-hash.png)

table是一个数组，其中的每个元素都指向一个entry对象，entry即哈希节点，结构如下：

```cpp
typedef struct dictEntry {
  //键
  void *key;
  // 值
  union {
    void *val;
    uint64_t u64;
    int64_t s64;
  } v;
  // 指向下个哈希表节点，形成链表
  struct dictEntry *next;
} dictEntry;
```
key保存着键值对中的键，而v属性则保存着键值对中的值，其中v可以是一个指针，也可以是一个uint64_t，或int64_t。
next指针指向下一个节点，用以解决哈希冲突。

![非空哈希表](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/non-empty-hash.png)

### 字典

```cpp
typedef struct dict {
  //类型特定函数
  dictType *type;
  // 私有数据
  void *privdata;
  // 哈希表
  dictht ht[2];
  // rehash 索引
  // 当rehash不在进行时，值为-1
  int rehashidx; /* rehashing not in progress if rehashidx == -1 */
} dict;
```
type属性和privdata属性是针对不同类型的键值对，为创建多态字典而设置的:
* type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用 于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同的类型特定函 数。
* 而privdata属性则保存了需要传给那些类型特定函数的可选参数。

```cpp
typedef struct dictType {
  //计算哈希值的函数
  unsigned int (*hashFunction)(const void *key);
  //复制键的函数
  void *(*keyDup)(void *privdata, const void *key);
  //复制值的函数
  void *(*valDup)(void *privdata, const void *obj);
  //对比键的函数
  int (*keyCompare)(void *privdata, const void *key1, const void *key2);
  //销毁键的函数
  void (*keyDestructor)(void *privdata, void *key);
  //销毁值的函数
  void (*valDestructor)(void *privdata, void *obj);
} dictType;
```
ht是一个具有两个哈希表元素的数组，一般情况下，字典只使用ht[0]哈希表，ht[1]只会对ht[0]进行rehash时使用。rehashidx记录了rehash的进度，如果目前没有在rehash，它的值为-1。

## 哈希算法
要将一个新的键值对添加到字典，程序需要先根据键计算出哈希值和索引值，然后根据索引值，将包含键值对的哈希表节点放到哈希表数组的指定索引位置上。

Redis计算哈希值的算法：

```cpp
// 使用字典设置的哈希函数，计算键key 的哈希值
hash = dict->type->hashFunction(key);
// 使用哈希表的sizemask 属性和哈希值，计算出索引值
//根据情况不同，ht[x],可以是ht[0]或者ht[1]
index = hash & dict->ht[x].sizemask;
```

Redis使用MurmurHash2算法来计算哈希值，这种算法的有点是即使输入的键是有规律的，算法仍然能给出一个很好的随机分布性，并且算法的计算速度也很快。

上面计算index值，使用哈希值和sizemask做了位运算&运算，这样做的原因是，由于sizemask永远等于size-1，因此`hash & dict->ht[x].sizemask`的值和`hash%dict->ht[x].size`的值是相等的，而且，由于使用的是位运算，因此速度比直接用取模运算更快。

## 解决哈希冲突

当有两个或多个键被分配到了哈希表数组的同一个索引位置上，我们这些键发生了哈希冲突。

Redis的哈希表使用链地址法来解决哈希冲突，每个哈希节点都有一个next指针，多个哈希节点可以使用next指针来构成一个单向链表，被分配到统一索引位置上的节点可以用这个单向链表链接起来，这就解决了哈希冲突的问题。这里使用的算法和Java中HashMap的实现是同一种方法。

因为dictEntry节点组成的链表没有指向链表表尾的指针，所以为了速度考 虑，程序总是将新节点添加到链表的表头位置(复杂度为O(1))，排在其他已有 节点的前面。

## rehash
随着操作的不断执行，哈希表里面的键值对会不断的增加或减少，为了让哈希表的负载因子维持在一个合理的范围，当哈希表保存的键值对数量太多或太少时，程序需要对哈希表的大小进行拓展或收缩。

Redis对字典哈希表rehash的步骤如下：

1. 为字典的ht[1]哈希表分配空间，这个哈希表的大小取决于要执行的操作以及ht[0]当前包含的键值对数量（即ht[0].used的值）
    - 如果是扩展操作，那么ht[1]的大小为第一个大于等于ht[0].used*2的2^n(2的n次幂)
    - 如果是收缩操作，那么ht[1]的大小为第一个大于等于ht[0].used的2^n
2. 将保存在ht[0]中的所有键值对rehash到ht[1]上面:rehash指的是重新计 算键的哈希值和索引值，然后将键值对放置到ht[1]哈希表的指定位置上。
3. 当ht[0]包含的所有键值对都迁移到了ht[1]之后(ht[0]变为空表)，释放 ht[0]，将ht[1]设置为ht[0]，并在ht[1]新创建一个空白哈希表，为下一次 rehash做准备。

> 哈希表的扩展与收缩
> 当以下条件中的任意一个被满足时，程序会自动开始对哈希表执行扩展操作:
> 1) 服务器目前没有在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的 负载因子大于等于1。
> 2) 服务器目前正在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负 载因子大于等于5。
>
> 哈希表的负载因子计算公式：
> 
> `load_factor = ht[0].used / ht[0].size`

## 渐进式Rehash
扩展或收缩哈希表需要将ht[0]里面的所有键值对rehash到ht[1] 里面，但是，这个rehash动作并不是一次性、集中式地完成的，而是分多次、渐进 式地完成的。

这样做的原因在于，如果ht[0]里只保存着四个键值对，那么服务器可以在瞬间 就将这些键值对全部rehash到ht[1];但是，如果哈希表里保存的键值对数量不是 四个，而是四百万、四千万甚至四亿个键值对，那么要一次性将这些键值对全部 rehash到ht[1]的话，庞大的计算量可能会导致服务器在一段时间内停止服务。

因此，为了避免rehash对服务器性能造成影响，服务器不是一次性将ht[0]里 面的所有键值对全部rehash到ht[1]，而是分多次、渐进式地将ht[0]里面的键值对 慢慢地rehash到ht[1]。

渐进式rehash的步骤：

1. 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表。
2. 在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示 rehash工作正式开始。
3. 在rehash进行期间，每次对字典执行添加、删除、查找或者更新操作时， 程序除了执行指定的操作以外，还会顺带将ht[0]哈希表在rehashidx索引上的所有 键值对rehash到ht[1]，当rehash工作完成之后，程序将rehashidx属性的值增 一。
4. 随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会 被rehash至ht[1]，这时程序将rehashidx属性的值设为-1，表示rehash操作已完 成。