---
title: 深入理解express和koa中间件模型
date: 2023-01-23 13:27:28
tags: [nodejs, express, koa, koa2]
categories:
  - 技术博客
  - 原创
---


express和koa2是nodejs常用的两个web框架，他们也都有自己的中间件模型。
我们都听说express的中间件模型是**线性模型**，而koa2的中间件模块是**洋葱模型**。
可对于这里面的细节，到底了解多少呢？

<!--more-->

# express

## 几个例子
### 示例1
都说express中间件是**线性模型**，请看如下代码：
```js
function m1(req, res, next) {
    console.log('中间件m1开始...');
    req.custom_params = {
        from_m1: '中间件m1设置的参数',
    };
    next();
    console.log('中间件m1结束...');
}
function m2(req, res, next) {
    console.log('中间件m2开始...');
    console.log(req.custom_params);
    req.custom_params = {
        ...req.custom_params,
        from_m2: '中间件m2设置的自定义参数',
    };
    next();
    console.log('中间件m2结束...');
}
function m3(req, res, next) {
    console.log('中间件m3开始...');
    console.log(req.custom_params);
    req.custom_params = {
        ...req.custom_params,
        from_m3: '中间件m3设置的自定义参数',
    }
    next();
    console.log('中间件m3结束...');
}
app.use([m1, m2, m3]);
app.use('/', (req, res) => {
    console.log('路由处理函数开始。。。');
    console.log(req.custom_params);
    res.send('HelloWorld!');
    console.log('路由处理函数结束。。。');
});


```


日志输出：

![Pasted image 20230323104916](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323104916.png)

我们定义了三个中间件 m1, m2, m3，并按顺序注册。每个中间件只是简单的打了一些日志并设置了一些自定义参数。
最终的结果如上图所示，每个中间件的处理过程都用不同颜色的框标注了出来。

看结果，每个中间件执行时先执行 `next()` 之前的逻辑，然后执行 `next()` 将控制权交给下一个中间件，下一个中间件处理完毕，再执行 `next()` 后面的逻辑。
这不就是koa的 **洋葱模型** 嘛？为什么还说express的中间件是线性模型？

### 示例2

我们改一下中间件m1的逻辑，在执行完 `next()` 后，通过 `res.send()` 返回一些数据看看：
```ts
function m1(req, res, next) {
    console.log('中间件m1开始...');
    req.custom_params = {
        from_m1: '中间件m1设置的参数',
    };
    next();
    console.log('中间件m1结束...');
    res.send('send from m1');
}
```

执行结果：
![Pasted image 20230323105935](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323105935.png)
浏览器结果：
![Pasted image 20230323110017](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323110017.png)

浏览器能够正常返回路由处理函数返回的数据，而且日志的前面和原来也一样，重点是日志的后面报错了，意思是说当已发送数据给客户端后就不能再设置headers了，也就是说在路由处理函数中已调用 `res.send()` 后，m1中间件再执行 `res.send()` 就会报错。

###示例3

那，如果我们把m1中间件的 `res.send()` 放到 `next()` 之前呢？
```js
function m1(req, res, next) {
    console.log('中间件m1开始...');
    req.custom_params = {
        from_m1: '中间件m1设置的参数',
    };
    res.send('send from m1');
    next();
    console.log('中间件m1结束...');
}
```

最终执行结果如下：
![Pasted image 20230323110511](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323110511.png)

浏览器结果：
![Pasted image 20230323110526](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323110526.png)

这个时候浏览器接收到的数据是中间件m1返回的数据。但从日志看来，即使m1中间件调用 `res.send()` 返回给客户端数据后，依然调用了后面的中间件，只不过是后面中间件无法再调用 `res.send()` 给客户端返回数据了。

其实express中间件的**线性模型**指的是对于`next()`的调用是线性的。

它的调用大致如下：
```js
app.use(function m1(req, res, next) {
  console.log('中间件m1开始...');
  req.custom_params = {
    from_m1: '中间件m1设置的参数',
  };
  (function m2(req, res, next) {
    console.log('中间件m2开始...');
    console.log(req.custom_params);
    req.custom_params = {
      ...req.custom_params,
      from_m2: '中间件m2设置的自定义参数',
    };
    (function m3(req, res, next) {
      console.log('中间件m3开始...');
      console.log(req.custom_params);
      req.custom_params = {
        ...req.custom_params,
        from_m3: '中间件m3设置的自定义参数',
      }
        (function handler(req, res, next) {
          console.log('路由处理函数开始...');
          console.log(req.custom_params);
          res.send('hello world!')
          console.log('路由处理函数结束...');
        })();
      console.log('中间件m3结束...');
    })();
    console.log('中间件m2结束...');
  })();
  console.log('中间件m1结束...');
});
```

就是说express中间件的原理就是一层一层函数的嵌套，m1调用next()时，会继续执行中间件m2，m2调用next()时，会继续执行路由处理函数，最后路由处理函数处理逻辑后，通过`res.send('hello world')`返回给接口字符串“hello world”。

###示例4

上面在中间件m1上，提前调用 `res.send()` 发送数据给客户端会导致后面中间件再发送`res.send()`时报错，那如果是调用`res`的其他方法呢？

比如，我们在调用`next()`之前通过`res.append()`方法给response添加自定义header：
```js
function m1(req, res, next) {
    console.log('中间件m1开始...');
    req.custom_params = {
        from_m1: '中间件m1设置的参数',
    };
    res.append('token', 'abcd');
    next();
    console.log('中间件m1结束...');
}
```
终端日志不报错。客户端得到的结果：
![Pasted image 20230323112001](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323112001.png)
从结果看，中间件m1在调用`next()`将控制权交给下一个中间件之前，可以通过`res.append()`给response添加自定义的header。

###示例5

那如果是在`next()`之后呢？
```js
function m1(req, res, next) {
    console.log('中间件m1开始...');
    req.custom_params = {
        from_m1: '中间件m1设置的参数',
    };
    next();
    res.append('token', 'abcd');
    console.log('中间件m1结束...');
}
```

![Pasted image 20230323112303](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323112303.png)

浏览器返回的结果：
![Pasted image 20230323112500](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323112500.png)

浏览器能正常接收到路由处理函数返回的数据，但是中间件m1设置的header头并未正确返回，而且看终端报错就是，和上面一样，在已发送数据给客户端后无法再设置headers。

## 总结
对比如上几个示例，我们可以总结express中间件如下特点：

1. express中间件按注册顺序进行调用
2. 中间件对于req,res的操作，只能在调用`next()`之前。即中间件对于请求的控制权仅仅是在`next()`之前，虽然`next()`之后的代码还是会继续执行，但对请求已无控制权，仅仅也只是执行代码而已。
3. `res.send()`这种返回数据给客户端的方法，只能在最后一个中间件处理。如果在前面中间件处理，那么后面的中间件的处理就无效了，在正常应用中肯定只能在最后一个中间件处理。
4. 其他诸如`res.append()`添加response headers的方法，在前置中间件中必须在调用`next()`之前处理，之后处理无效，并报错。

所以，express中间件的线性可以用如下图描述：

![Pasted image 20230323114425](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323114425.png)



# koa
koa是在express之后出来的框架，使用了es6的[[Async&Await]]，其中间件模型被称为**洋葱模型**。

![Pasted image 20230225232756](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230225232756.png)

## 几个例子

### 示例1

```js
async function m1(ctx, next) {
  console.log('中间件m1开始执行。。。');
  ctx.request.custom_params = {
    from_m1: '中间件m1设置的参数',
  }
  next();
  console.log('中间件m1执行结束。。。');
}

async function m2(ctx, next) {
  console.log('中间件m2开始执行。。。');
  console.log(ctx.request.custom_params);
  ctx.request.custom_params = {
    ...ctx.request.custom_params,
    from_m2: '中间件m2设置的参数'
  }
  next();
  console.log('中间件m2执行结束。。。');
}
async function m3(ctx, next) {
  console.log('中间件m3开始执行。。。');
  console.log(ctx.request.custom_params);
  ctx.request.custom_params = {
    ...ctx.request.custom_params,
    from_m3: '中间件m3设置的参数'
  }
  next();
  console.log('中间件m3执行结束。。。');
}

app.use(m1);
app.use(m2);
app.use(m3);

app.use(async ctx => {
  console.log('路由处理函数开始执行。。。');
  ctx.body = "Hello KOA!";
  console.log('路由处理函数执行结束。。。');
});

```

日志输出：
![Pasted image 20230323115746](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323115746.png)

第一个例子和express的示例1差不多，从日志上来看，感觉处理过程也是差不多的。

express中主要对req，res的处理过程，那么我们在koa中对req，res进行处理看一下。

###示例2

我们改一下中间件m1和m2的代码，改为如下：
![Pasted image 20230323120455](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323120455.png)

在中间件m1调用`next()`之前设置一下header，加入token="token from m1"，在m2中打印一下response的header['token']，日志输出如下：
![Pasted image 20230323120649](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323120649.png)
![Pasted image 20230323120708](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323120708.png)
从日志输出以及浏览器结果来看，中间件m1在调用`next()`之前，对response的操作是有效的。

那，我们将中间件m1的设置response的header放到调用`next()`之后呢？

###示例3
![Pasted image 20230323121002](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323121002.png)

我们将m1设置response header放到调用`next()`之后，在m2中继续打印一下response的相关header，结果如下：
![Pasted image 20230323121131](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323121131.png)

![Pasted image 20230323121149](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323121149.png)

从日志上看，中间件m2好像并为获取到m1设置的header，但是从实际浏览器最终获得的返回数据看，m1的设置确实是成功了，浏览器能正确拿到m1设置的header。
因为m1中设置response header是在调用`next()`之后，也就是后面的中间件都执行完了才设置的，因此m2的执行是在设置header之前，所以m2是肯定拿不到相关的header的。

这里就是koa中间件和express中间件不同的地方，也是理解koa **洋葱模型** 的关键。

###示例4
我们在m1和m2中间件调用`next()`之后，修改返回的body会发生什么呢？

![Pasted image 20230323124613](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323124613.png)
我们在中间件m1调用`next()`之后，先打印`response.body`，然后再重新设置`response.body='Hello From m1'`，在中间件m2调用`next()`之后，设置`response.body='Hello From m2'`，得到结果如下：

![Pasted image 20230323124901](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323124901.png)

从日志可以看出，m1中打印出了m2设置的body，而且经过m1重新设置后，浏览器得到的数据是m1设置的'Hello From m1'。

###示例5
我们将m1和m2设置请求参数的语句顺序调整一下，放到调用`next()`之后：

![Pasted image 20230323125513](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323125513.png)
m1在调用`next()`之后，打印request的自定义参数，m2在调用`next()`之后，设置自定义参数`from_m2='中间件m2设置的参数'`，结果如下：

![Pasted image 20230323125644](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323125644.png)

从结果看，在调用`next()`之后，还是可以操作request，并且结果会在上一个中间件调用`next()`之后获得。

## 总结
从上面几个koa的示例，我们可以看出koa中间件**洋葱模型**的一些特点：
1. 每个中间件会有两次处理req,res的机会，分别是在调用`next()`前后
2. 每个中间件处理req,res的机会顺序正好相反
3. 调用`next()`会将控制权交给下一个中间件
4. 整个流程看起来就像是洋葱圈，请求先从最外面的中间件进行，逐步向内。返回时，由最内侧中间件，逐步向外。
5. 中间件不管处于什么位置，对res都有操作权，流程后面的操作会覆盖前面的操作，而不像express一样会报错。

![Pasted image 20230323130313](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323130313.png)


# 对比总结一下
所谓的**线性模型**和**洋葱模型**其实说的是对请求的控制操作的流程。

express中间件模型示意图：
![Pasted image 20230323114425](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323114425.png)

koa中间件模型示意图：
![Pasted image 20230323130313](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/Pasted%20image%2020230323130313.png)

- express的线性模型只能是在调用`next()`之前进行，一旦调用`next()`便将控制权交给下个中间件，直至最后一个中间件，最终由最后一个中间件返回给客户端数据
- koa的洋葱模型是每个中间件有两次操作请求响应的机会，分别是在调用`next()`前后。这两次操作请求响应的顺序正好相反，如同请求穿过洋葱一般，先由外向里，然后再由里向外返回来。
