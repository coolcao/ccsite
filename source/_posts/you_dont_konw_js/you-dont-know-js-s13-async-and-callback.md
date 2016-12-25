---
title: 学习笔记-你不知道的js-异步和回调
date: 2016-12-25 21:46:52
tags: [js]
categories:
- 学习笔记
- 你不知道的js
---

Javascript引擎并没有时间的概念，只是一个按需执行javascript任意代码片段的环境。“事件”调度总是由包含它的环境运行。

什么是事件循环，先通过一段伪代码了解下：

```js
//eventLoop是一个用作队列的数组
var eventLoop = [];
var event ;
while(true){
	if(eventLoop.length > 0){
		event = eventLoop.shift();
		try{
			event();
		}catch(){
			reportError(err);
		}
	}
}
```

这是一个极度精简的代码，用来说明概念。
有一个用while循环实现的持续运行的循环，循环的每一轮称为一个tick。对每个tick而言，如果在队列中有等待事件，那么就会从队列中摘下一个事件并执行。这些事件就是你的回调函数。

`setTimeout()`并没有把你的回调函数挂在事件循环队列中。它所做的是设定一个定时器。当定时器到时后，环境会把你的回调函数放在事件循环中，这样在未来的某个tick会摘下执行这个回调。

如果这时候事件循环中已经有20个项目了怎么办？你的回调就会**等待**。它得排到其他项目后面--通常没有抢占方式支持直接将其排到队首。这也解释了为什么`setTimeout()`定时器的精度可能不高。

考虑一个需要遍历很长的结果列表进行值转换的ajax响应处理函数：
```js
var res = [];
function response(data){
	res = res.concat(data.map(function(val){
		return val * 2;
	}));
}
```
如果有1000万条记录的话，就可能需要运行相当一段时间了。这样，页面上的其他代码都不能运行，包括不能有其他的response()调用或UI刷新，甚至是像滚动，输入，按钮点击这样的用户事件。这是相当痛苦的。
这里给出一个非常简单的方法：
```js
var res = [];
function response(data){
	var chunk - data.splice(0,1000);
	res = res.concat(chunk.map(function(val){
		return val * 2;
	}));

	if(data.length > 0){
		setTimeout(function(){
			response(data);
		},0);
	}
}
```
我们把数据集合放在最多包含1000条项目的块中。这样我们就确保了“进程”运行时间很短，即使这意味着需要更多的后续“进程”，因为事件循环队列的交替运行会提高站点的响应。
当然，我们并没有协调这些“进程”的顺序，所以结果的顺序是不可预测的。

> 严格来说，setTimeout(...,0)并不直接把项目插入到事件循环队列。定时器会在有机会的时候插入事件。举例来说，两个连续的setTimeout(...,0)调用不能保证会严格按照调用顺序处理，所以各种情况都有可能出现，比如定时器漂移，在这种情况下，这些事件的顺序就不可预测。在Nodejs中，类似的方法是process.nextTick()。尽管它们使用方便（通常性能也很高），但并没有（至少到目前为止）直接的方法可以适应所有环境来确保异步事件的顺序。
