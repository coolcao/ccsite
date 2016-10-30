---
title: js数组拷贝
date: 2016-10-30 23:14:52
tags: [js,Array]
categories: 技术博客
---

利用js原生已实现的方法，我们就可以不用自己写循环实现数组的拷贝复制。

原数组：
```js
let array1 = [1,'a',true,null,undefined];
```

<!--more-->

## slice()方法
```js
let c1 = array1.slice();
```

## concat()方法
```js
let cc1 = array1.concat();
```

## from()方法
```js
let fc1 = Array.from(array1);
```

## push()方法
```js
let pc1 = [];
Array.prototype.push.apply(pc1,array1);
```

## map()方法
```js
let mc1 = array1.map(function(item){
    return item;
});
```

以上几种方法都能实现数组的`浅拷贝`，即数组的每一项只能是原始类型的数据，如果数组的项包含引用类型，如数组（即js中的二维数组），对象等，以上方法复制的项只是引用。
还有一种方法是，使用json进行转换，先将数组序列化为json字符串，然后再将字符串转换成json对象即可。
## JSON转换
```js
let jsonc = JSON.parse(JSON.stringify(array1));
```
这种方法可以变相的实现`深拷贝`,但是这种方法也有其限制：
* 首先，数组中的项如果是`undefined`，那么转换后将变为`null`
* 如果数组的项为对象，那么对象之间不可相互引用。会造成循环引用，无法JSON序列化。
