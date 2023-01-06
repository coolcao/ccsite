---
title: 前缀树结构做路由匹配
date: 2021-03-06 10:34:27
tags: [算法与数据结构, 前缀树]
categories:
  - 技术博客
  - 原创
---

不管是做前端还是后端，都可能会遇到路由匹配的需求。如果是静态路由，可以直接用哈希表进行存储，查找时直接从哈希表查即可，速度非常快，复杂度 O(1)。
但在实际场景中，更多的是动态路由的匹配，动态路由直接用哈希表就有点力不从心了。动态路由可以用*前缀树*这个结构。

<!-- more -->

## 前缀树

前缀树（Trie 树）是一种树形结构，经常被用作单词前缀补全以及动态路由的匹配实现。

前缀树的特点：

- 根节点不包含字符，除根节点外的每一个子节点都包含一个字符。
- 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串
- 每个节点包含的所有子节点包含的字符不同
- 从第一字符开始有连续重复的字符只占用一个节点

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1672977233_20230106110544490_319631237.png)
上图是一棵前缀树，表示了关键字集合 {"a", "to", "tea", "ten", "i", "in", "inn"}。

### 前缀树的基本实现

```ts
// Trie节点
class TrieNode {
  isWord: boolean;
  next: Map<string, TrieNode>;
  constructor() {
    this.isWord = false;
    this.next = new Map();
  }
}

class TrieTree {
  root: TrieNode;
  constructor() {
    this.root = new TrieNode();
  }

  insert(word: string) {
    let cur = this.root;
    for (const c of word) {
      if (!cur.next.has(c)) {
        cur.next.set(c, new TrieNode());
      }
      cur = cur.next.get(c);
    }
    cur.isWord = true;
  }
  search(word: string) {
    let cur = this.root;
    for (const c of word) {
      if (!cur.next.has(c)) {
        return false;
      }
      cur = cur.next.get(c);
    }

    return cur.isWord;
  }
}
```

## 动态路由实现

基于前缀树这个结构，我们可以应用于路由的匹配。
比如下面这张图可以匹配如下的路由规则：

- GET /users
- GET /users/:id
- GET /about
- GET /blogs
- GET /blogs/:id
  ![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1672977235_20230106114139823_1185599112.png)

其中根节点是 http 请求方法，每个子节点都是路由中的一部分。
后面两个 `:id` 节点是通配符，可以匹配动态路由。

所以每个节点的结构如下：

```ts
class RouterNode {
  path: string; // 完整路径，只存于最后一个节点，表示一个完整的路径
  part: string; // 路由中由 '/' 分隔的部分
  children: Map<string, RouterNode>; // 子节点
  isWild: boolean; // 是否是通配符节点
  level: number; // 层级，从0开始
  constructor(
    path?: string,
    part?: string,
    isWild?: boolean,
    level: number = 0
  ) {
    this.part = part;
    this.path = path;
    this.isWild = isWild;
    this.children = new Map();
    this.level = level;
  }
}
```

结构中，path 字段表示完整的路径，只存于最后一个节点，表示一个完整的路径。part 表示路径中由 `/` 分隔的每一部分。children 存储的是子节点。isWild 标识该节点是否是通配符，level 标识节点的层级，在进行路由匹配时，层级要和路由中的 part 层级一致。

对于路由匹配功能来说，除了要存储路径这个前缀树，还要存储路由的处理函数，因此我们的路由 Router 大致结构如下：

```ts
class Router {
  // 前缀树保存节点信息
  root: Map<string, RouterNode>;
  // 保存路由信息
  route: Map<string, Function>;
  constructor() {
    this.root = new Map();
    this.route = new Map();
  }

  // 添加路由
  addRoute(method: string, path: string, handler: Function) {
    if (!this.root.has(method)) {
      this.root.set(method, new RouterNode());
    }

    let root = this.root.get(method);

    const key = method + path;

    const parts = parsePath(path);

    for (const [i, part] of parts.entries()) {
      if (!root.children.has(part)) {
        root.children.set(
          part,
          new RouterNode(null, part, part[0] == ":" || part[0] == "*", i + 1)
        );
      }
      root = root.children.get(part);
    }
    // path只存于路径的最后一个节点
    root.path = path;
    this.route.set(key, handler);
  }

  // 查找路由，返回对应的节点和参数字典
  getRoute(method: string, path: string) {
    const params = new Map();
    const searchParts = parsePath(path);
    if (!this.root.has(method)) return null;

    let node = this.root.get(method);

    for (const [i, part] of searchParts.entries()) {
      let tmp = "";
      for (const child of node.children.values()) {
        // 只匹配相同层级的
        if (child.level == i + 1) {
          // 路由匹配上，或者遇到通配符
          if (child.part == part || child.isWild) {
            if (child.isWild && child.part[0] == ":")
              params[child.part.slice(1)] = part;
            tmp = child.part;
          }
        }
        if (child.level > i + 1) {
          break;
        }
      }
      // 星号通配符直接返回
      if (tmp[0] == "*") {
        const key = method + node.children.get(tmp).path;
        return { handler: this.route.get(key), params };
      }

      node = node.children.get(tmp);
    }

    const key = method + node.path;
    return { handler: this.route.get(key), params };
  }
}
```

如上就基于前缀树实现了简单地动态路由匹配功能。
在使用时先添加路由，然后通过`getRoute()`方法就能查询到该路由对应的处理函数以及路由中的参数。

```ts
router.addRoute("GET", "/blogs/*", function () {
  console.log("这是请求博客所有内容接口");
});
router.addRoute("GET", "/users/:id", function () {
  console.log("用户详情接口");
});
router.addRoute("GET", "/users", function () {
  console.log("用户列表接口");
});
router.addRoute("GET", "/tasks", function () {
  console.log("任务列表接口");
});
router.addRoute("GET", "/tasks/:id", function () {
  console.log("任务详情接口");
});

const r = router.getRoute("GET", "/blogs/5/4/3");
if (r) {
  r.handler ? r.handler() : console.log("未匹配到路由");
}
```
