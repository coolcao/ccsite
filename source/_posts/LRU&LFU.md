---
title: LRU算法和LFU算法.md
date: 2020-09-14 15:17:39
tags: [算法, LRU, LFU]
categories:
- 技术博客
- 原创
---

LRU和LFU是两个缓存淘汰策略算法，其本身的思想很简单，但却非常有意思。

而且也是面试中经常考察的算法，虽然其思想很简单，但却能考察程序员对基础数据结构的理解与应用。

这篇文章，我们就来看一下这两个算法，并一步一步解析实现了他。

<!-- more -->

## LRU算法
LRU即Least Recent Used，最近最少使用算法。其思想就是，如果空间不够了，那么要淘汰的是，最近最少使用的元素。

我们来看一下LeeCode上的原题：

```
运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。
它应该支持以下操作： 
    - 获取数据 get 。
    - 写入数据 put 。

获取数据 get(key) : 
    - 如果关键字 (key) 存在于缓存中，则获取关键字的值（总是正数），否则返回 -1。
写入数据 put(key, value) : 
    - 如果关键字已经存在，则变更其数据值；
    - 如果关键字不存在，则插入该组「关键字/值」。
    - 当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

进阶:
你是否可以在 O(1) 时间复杂度内完成这两种操作？

示例:
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );
cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得关键字 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得关键字 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/lru-cache
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

这是LeetCode上的一个考察LRU算法的题目，简单粗暴，设计并实现LRU缓存。

为了简化模型，这里key/value都将设计成int类型，且value总是正数，如果get(key)不存在时，返回-1。
这里有个限制，也是进阶的要求：**get和put的时间复杂度为O(1)**

首先，如果要求时间复杂度为O(1)，那么我们首先想到的是使用哈希表这个结构。虽然数组也能实现O(1)，但数组必须通过下标才能访问，因此其使用场景很受限。也就是说，key/value的映射，这里我们要使用哈希表来实现。

再来往下分析，如果容量已满，我们要删除最近最少使用的元素，也就是说，我们还要记录元素访问时间的顺序。所以这里应用用一个线性结构，而且插入删除元素的复杂度为O(1)，那这里只有链表这一结构。而且得使用双向链表，因为在删除元素时，要获取其前驱与后驱。

分析到这里，大致用到的数据结构就有了：哈希表做键值映射，双向链表结构来记录元素访问的时间顺序。

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1609321221_20200914161746014_601475315.png)

大致的结构如上图，哈希表用来做key/value映射，下面的链表，有一个head指针和一个tail指针，我们约定head是最久访问，tail是最近访问。

当一个元素被访问时，随即被调整到链表的tail尾部。当容量已满，需要移除最久访问元素时，直接移除head元素即可。

最终实现的代码：

```go
// 双向链表节点
type node struct {
	key  int
	val  int
	pre  *node
	next *node
}

func moveToTail(head *node, tail *node, n *node) (*node, *node) {
	if head == nil && tail == nil {
		return head, tail
	}
	// 已经是尾节点，不需要再调整
	if n == tail {
		return head, tail
	}

	// 头节点，移动到尾部并调整head指针，tail指针
	if n == head {
		head = head.next
		head.pre = nil

		tail.next = n
		n.pre = tail
		tail = tail.next
		tail.next = nil
		return head, tail
	}

	pre, next := n.pre, n.next
	n.pre, n.next = nil, nil
	pre.next = next
	next.pre = pre

	tail.next = n
	n.pre = tail
	tail = tail.next
	return head, tail

}

func removeHead(head *node, tail *node) (*node, *node) {
	if head == tail {
		return nil, nil
	}
	next := head.next
	head.next = nil
	head = next
	head.pre = nil
	return head, tail
}

func addToTail(head, tail, n *node) (*node, *node) {
	if head == nil && tail == nil {
		head = n
		tail = n
		return head, tail
	}
	tail.next = n
	n.pre = tail
	tail = tail.next
	return head, tail
}

type LRUCache struct {
	capacity int           // 容量
	used     int           // 已使用
	data     map[int]*node // 哈希表数据域
	head     *node         // 链表头节点，标识最先入缓存的节点
	tail     *node         // 链表尾节点，标识最后入缓存的节点
}

func Constructor(capacity int) LRUCache {
	cache := LRUCache{
		capacity: capacity,
		used:     0,
		data:     map[int]*node{},
		head:     nil,
		tail:     nil,
	}
	return cache
}

func (this *LRUCache) Get(key int) int {
	n, ok := this.data[key]
	if !ok {
		return -1
	}

	// 存在节点，调整其在链表中的位置
	this.head, this.tail = moveToTail(this.head, this.tail, n)

	return n.val
}

func (this *LRUCache) Put(key int, value int) {
	n, ok := this.data[key]
	if !ok {
		// 不存在
		// 判断容量是否已满
		n := &node{val: value, key: key}
		if this.capacity == this.used {
			// 已满
			// 删除头节点，并将新节点添加到尾部
			headKey := this.head.key
			delete(this.data, headKey)
			this.head, this.tail = removeHead(this.head, this.tail)
			this.head, this.tail = addToTail(this.head, this.tail, n)
		} else {
			// 未满
			this.head, this.tail = addToTail(this.head, this.tail, n)
			this.used++
		}
		this.data[key] = n
	} else {
		// 存在，更新
		n.val = value
		this.head, this.tail = moveToTail(this.head, this.tail, n)
		this.data[key] = n
	}
}

```

在data里，我们不是直接使用key和value的映射，而是key和链表节点指针的映射。这样做的好处是，我们把所有的处理都当作对链表节点的处理，逻辑上更简便。当然，缺点就是，效率上会比直接key和value的映射要低，虽然其复杂度也是O(1)。

## LFU算法

