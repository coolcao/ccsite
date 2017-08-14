---
title: js中模拟实现私有属性
date: 2017-08-14 15:14:50
tags: [js]
categories:
- 技术博客
- 原创
---

今天在看《你不知道的js》这本书时，无意看到Object还有个方法叫做 `getOwnPropertySymbols()` ，用来获取对象的Symbol属性。记得之前看过一些文章说，可以使用Symbol来实现私有属性，如果能直接使用 `getOwnPropertySymbols()` 方法获取Symbol属性，那还是私有属性么？

今天复习整理一下关于js中创建私有属性的一些问题。

由于js并不是Java那种类式面向对象，因此即使es6添加了class支持，js根上还是基于原型的面向对象，不支持什么私有，公有属性的。

要想实现私有属性，基于现有的js，途径只有一个： 闭包。

<!-- more -->

至于什么是闭包，网上一大把，有兴趣也可以看看我之前整理的[文章](http://coolcao.com/2017/02/06/closure/)。

## 使用原始的闭包
```js
const User = (function () {
    return class User{
        constructor(name) {
            this.getName = function () {
                return name;
            }
            this.setName = function (name) {
                this.getName = function () {
                    return name;
                }
            }
        }
    };
})();
```

这种方式的确可以实现私有的属性，而且如果有子类继承，也可以如下写法：
```js
const Boy = (function () {
    return class Boy extends User{
        constructor(name){
            super(name);
            this.getGender = function () {
                return 'boy';
            }
        }
    }
})();
```

但问题也有：
* 代码组织混乱。由于对构造器函数形成一个闭包，因此所有的setter,getter函数都写在了构造器内。可以从上面的User的写法中看出，setter函数中，还要再定义一遍getter，这中混乱，不是一般能忍受的。如果业务逻辑中还有其他的更多操作，那么混乱程度一下子就上来了。
* 闭包的内存开销不容小觑。

针对上面的问题，我们可不可以将name单独拿出来放到单独一个地方存储，优化一下代码的组织呢？比如下面这个：

```js
const User = (function () {
    let privateName = null;
    return class User{
        constructor(name) {
            privateName = name;
        }
        getName(){
            return privateName;
        }
        setName(name) {
            privateName = name;
        }
    };
})();
```

上面这个例子，貌似是可以的，这样代码也清晰了，也能实现私有属性。

但是，仔细分析一下，真的可以么？

如此的话，是不是所有的实例对象都共享一个 privateName 属性？后面的实例会覆盖前面实例的值。看下面：

```js
const u1 = new User('coolcao');
const u2 = new User('lili');
console.log(u2.getName());
console.log(u1.getName());
// lili
// lili
```

针对上面的问题，我们使用一个 privateName 保存私有属性，会被覆盖，那么我们如果使用一个数组，保存多个，然后针对每个实例，生成一个唯一的存取标识呢？

## 基于散列实现
```js
const User = (function () {
    const privateData = {};
    let i = 0;
    return class User {
        constructor(name){
            this['_id'] = i ++;
            privateData[this['_id']] = {name: name};
        }
        getName() {
            return privateData[this['_id']].name;
        }
        setName(name) {
            privateData[this['_id']].name = name;
        }
    };
})();
```

我们使用 privateData 这个对象来保存所有实例的私有属性，针对每个实例，使用一个id进行标识，每实例化一个实例，该id会自动加1。然后将该id作为键值，将私有属性存入 privateData 对象。
嗯，这样看上去比直接在构造器中使用闭包要清晰多了，而且实例与实例之间也不冲突。

在es6之前，这可能是最合适的方案。虽然也会存在问题：

每个实例对象都会引用 privateData ，因此，还是由于闭包的问题，如果实例太多的话，内存是个问题。

## WeakMap实现

幸好 es6 来了，带来了一个叫做 `WeakMap` 的东西，具体这东西是啥呢？可以看看[阮老师的教程](http://es6.ruanyifeng.com/#docs/set-map#WeakMap)

简单说来，WeakMap键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

好了，那么上面的这个强引用关系，我们可以使用 WeakMap 弱引用来代替：

```js
const User = (function () {
    const privateData = new WeakMap();
    class User {
        constructor(name) {
            privateData.set(this,{name:name});
        }
        getName() {
            return privateData.get(this).name;
        }
        setName(name) {
            privateData.get(this).name = name;
        }
    }
})();
```

如此的代码，干净清爽了许多，而且由于WeakMap是弱引用，如果没有其他引用和该键引用同一个对象,这个对象将会被当作垃圾回收掉。解决了内存泄露的问题。

好了，js模拟闭包，就这几个方式了，从这几个例子来看，都使用了自执行函数（IIFE），因此都会形成闭包。这也是我最开始说的，要想在js实现私有属性，只能使用闭包。

## Symbol的问题
话说回来我当初的疑问，ES6的Symbol实现的私有属性有啥问题呢？

看下面例子：

```js
const User = (function () {
    const NAME = Symbol('User#Name');
    return class User{
        constructor(name){
            this[NAME] = name;
        }
        getName(){
            return this[NAME];
        }
        setName(name) {
            this[NAME] = name;
        }
    }
})();
```

NAME是个Symbol，从外部并不能拿到NAME确切的值，好像是有点私有属性的意思。但是 有一个 `Object.getOwnPropertySymbols()` 方法可以拿到对象所有的Symbol属性，虽然我不知道具体存了个啥，但是能拿到这个标识，就可以修改属性值了：

```js
const smbs = Object.getOwnPropertySymbols(user);
for(let s of smbs) {
    user[s] = 'good';
}
console.log(user.getName());
// good
```
因此，严格意义上说，Symbol其实并不能实现私有属性。

不过倒是可以将上面第二种基于散列的方式改为Symbol的方式：

```js
const User = (function () {
    const privateData = {};
    return class User {
        constructor(name){
            this._name = Symbol('name');
            privateData[this._name] = {name: name};
        }
        getName() {
            return privateData[this._name].name;
        }
        setName(name) {
            privateData[this._name].name = name;
        }
    };
})();
```

这样对外暴露的只是 _name 的这个符号，外部还是无法直接访问每个实例的 privateData 中的值。但这个其实和第二种是一样的，闭包引起的问题还是无法解决。

## 参考
* [Private instance members with weakmaps in JavaScript](https://www.nczonline.net/blog/2014/01/21/private-instance-members-with-weakmaps-in-javascript/)
* [Private Properties(MDN)](https://developer.mozilla.org/en-US/Add-ons/SDK/Guides/Contributor_s_Guide/Private_Properties)
* [JavaScript实现私有属性](http://www.cnblogs.com/ihardcoder/p/4914938.html)
