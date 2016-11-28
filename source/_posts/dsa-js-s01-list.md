---
title: 学习笔记-数据结构与算法js实现-【2】列表
date: 2016-11-28 01:05:30
tags: [数据结构与算法,js]
categories:
- 学习笔记
- 数据结构与算法
---

列表是一组有序的数据，列表中每个数组项被称为元素。
不含任何元素的列表称为*空列表*。列表中包含的元素个数被称为列表的length。
可以在列表末尾append一个元素，也可以在一个给定的元素后或列表的起始位置insert一个元素。
使用remove方法从列表中删除元素，使用clear方法清空列表中所有的元素。

<!--more-->

使用getElement()方法显示当前元素。
列表拥有描述当前位置的属性。列表有前有后，分别对应front和end。使用next()方法可以从当前元素移动到下一个元素，使用prev()方法可以移动到当前元素的前一个元素。还可以使用moveTo()方法直接移动到指定位置。

## 方法列表
|方法或属性名称|类型|说明|
|----|----|----|
|pos|属性|当前位置|
|length()|方法|返回当前列表中元素的个数|
|clear()|方法|清空列表|
|getElement()|方法|返回当前位置元素|
|insert(n,e)|方法|在位置n插入元素e|
|append(e)|方法|在列表末尾添加元素|
|remove(e)|方法|从列表中删除元素|
|front()|方法|将列表的当前位置移动到第一个位置|
|end()|方法|将列表的当前位置移动到最后一个位置|
|prev()|方法|将当前位置移动到前一个位置|
|next()|方法|将当前位置移动到后一个位置|
|hasNext()|方法|是否有下一个元素|
|hasPrev()|方法|是否有前一个元素|
|currPos()|方法|返回当前位置|
|moveTo()|方法|将当前位置移动到指定位置|

## 定义List类
```js
class List{
  constructor(){
    this.pos = 0;   //当前位置
    this.data = []; //存储数据
  }
  //返回当前列表中元素的个数
  length(){
    return this.data.length;
  }
  //清空列表
  clear(){
    delete this.data;
    this.data = [];
    this.pos = 0;
  }
  //返回当前位置元素
  getElement(){
    return this.data[this.pos];
  }
  //插入元素
  insert(n,e){
    
  }
}
```