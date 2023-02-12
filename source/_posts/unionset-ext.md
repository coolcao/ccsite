---
title: 并查集拓展
date: 2022-10-20 18:18:33
tags: [算法与数据结构, 并查集]
categories:
- 技术博客
- 原创
---

在[并查集](/2022/10/18/unionset)的实现里面有提到，并查集两种实现方式：
- 一种是使用数组，索引作为元素，数组值作为集合，这种方式适合快速查询。
- 一种是基于森林实现，不过也是使用数组结构，索引作为元素，数组值存储其父节点的方式。

<!--more-->

在一般使用并查集的场景中，基本上都是采用第二种，基于森林的实现方式。

不过在真实业务场景中，元素可能不仅仅是数字类型，也有可能是字符串类型，或者对象类型。

所以，在实现上，需要换一种结构实现。


## 基于哈希表实现
如果元素是字符串或对象类型，我们可以直接使用Map代替数组来实现查并集。

其中元素作为Map的键key，而其所属的集合名作为Map的value值。

> 初始情况下，元素的所属集合（即parent节点）为自己。

```js
class UnionFindSet {
    constructor(names) {
        this.nameMap = new Map();
        if (names) {
            for (const name of names) {
                this.nameMap.set(name, name);
            }
        }

    }
    // 查找元素，属于哪个集合
    find(name) {
        let root = this.nameMap.get(name);
        while (root && root != this.nameMap.get(root)) {
            root = this.nameMap.get(root);
        }
        return root;
    }
	// 添加元素
    add(name) {
        let root = this.find(name);
        if (!root) {
            this.nameMap.set(name, name);
            root = name;
        }
        return root;
    }
    // 合并操作，将集合x和集合y合并成一个集合
    union(x, y) {
        const rootX = this.add(x);
        const rootY = this.add(y);
        if (rootX == rootY) {
            return false;
        }
        if (rootX < rootY) {
            this.nameMap.set(rootY, rootX);
        } else {
            this.nameMap.set(rootX, rootY);
        }

        return true;
    }

    // 查询操作：判断x和y是否同属于 一个集合
    isConnected(x, y) {
        return this.find(x) == this.find(y);
    }
}

```

## 基于对象节点
由于并查集是一个树的结构，因此，我们可以采用对象节点的方式来实现，就像实现二叉树一样，定义一个节点对象，然后通过parent指针指向其父节点，一次来达到树的节点引用。

```js
class ItemNode {
    constructor(val) {
        this.val = val;
        // 初始情况下，parent指向自己
        this.parent = this;
    }
}
class UnionFindSet {
    constructor(names) {
        this.map = new Map();
        if (names) {
            for (const name of names) {
                const node = new ItemNode(name);
                this.map.set(name, node);
            }
        }

    }
    // 查找元素，属于哪个集合
    find(name) {
        let node = this.map.get(name);
        while (node && node != node.parent) {
            node = node.parent;
        }
        return node;
    }
    add(name) {
        let root = this.find(name);
        if (!root) {
            const node = new ItemNode(name);
            this.map.set(name, node);
            root = node;
        }
        return root;
    }
    // 合并操作，将集合x和集合y合并成一个集合
    union(x, y) {
        const rootX = this.add(x);
        const rootY = this.add(y);
        if (rootX == rootY) {
            return false;
        }
        if (rootX.val < rootY.val) {
            rootY.parent = rootX;
        } else {
            rootX.parent = rootY;
        }

        return true;
    }

    // 查询操作：判断x和y是否同属于 一个集合
    isConnected(x, y) {
        return this.find(x) == this.find(y);
    }
}

```


此两种实现方式，在使用上可能更具通用性，不管要处理的数据类型是什么，都可以应对。