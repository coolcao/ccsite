---
title: NodeJs技术栈处理SSE
date: 2024-01-23 09:41:14
tags: [sse]
categories:
- 技术博客
- 转载
---

SSE(Server Send Events)本身并不是一个新技术，但随着近两年ChatGPT的爆火，使得SSE技术也重新火了一把。
对于SSE细节，可参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events)和 [阮一峰的博客](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)。

<!-- more-->

SSE 简单来说就是一种采用了固定格式的流式请求，允许服务器端以流的形式向客户端发送事件，其本质还是采用了流式的HTTP。

但SSE本身的一些特性也是限制了其应用场景：
1. SSE是单向通道，只能服务器向浏览器发送。
2. SSE默认只能使用GET方法，不允许其他如POST方法

## 限制
1. 只能服务器向浏览器发送，在具体实现上，`EventSource` 类在 `window` 对象上，因此，标准的SSE实现只能在浏览器端使用，而*无法在服务器端使用*
2. 标准SSE只能用GET方法，不能使用POST方法。如果一个POST接口采用了SSE向客户端返回数据，标准SSE下会无从下手。

## 使用支持POST的SSE
如果要支持POST的SSE，标准的 `EventSource` 无法做到，我们只能使用第三方的实现，这里推荐微软的 `fetch-event-source`库，其github地址为: [https://github.com/Azure/fetch-event-source](https://github.com/Azure/fetch-event-source)。

这是一个底层采用 `fetch` 实现的SSE客户端，支持POST方法。

> 但由于其采用的是 `fetch` 实现，所以它也只能在浏览器端使用，而不能在nodejs端使用。

```js
class RetriableError extends Error { }
class FatalError extends Error { }

fetchEventSource('/api/sse', {
    async onopen(response) {
        if (response.ok && response.headers.get('content-type') === EventStreamContentType) {
            return; // everything's good
        } else if (response.status >= 400 && response.status < 500 && response.status !== 429) {
            // client-side errors are usually non-retriable:
            throw new FatalError();
        } else {
            throw new RetriableError();
        }
    },
    onmessage(msg) {
        // if the server emits an error message, throw an exception
        // so it gets handled by the onerror callback below:
        if (msg.event === 'FatalError') {
            throw new FatalError(msg.data);
        }
    },
    onclose() {
        // if the server closes the connection unexpectedly, retry:
        throw new RetriableError();
    },
    onerror(err) {
        if (err instanceof FatalError) {
            throw err; // rethrow to stop the operation
        } else {
            // do nothing to automatically retry. You can also
            // return a specific retry interval here.
        }
    }
});
```

## NodeJs后端对接SSE接口
如果后端要对接第三方的SSE接口，目前没有现成的库可以使用，只能自己实现。
但我们知道其本质就是一个流式的HTTP请求，所以，其实现也不复杂，就是监听流式请求，解析成SSE协议的格式即可。

这里我们使用 `axios` 来解析。

```js
const response = await axios({
  method: 'POST',
  url: '/sse',
  data: {},
  responseType: 'stream'
});
const stream = response.data;
const decoder = new TextDecoder('utf-8');
stream.on('data', (data) => {
  const current = decoder.decode(data, { stream: true });
  // 解析一行为SSE事件
  const event = parse(current);
  ...
});
stream.on('end', () => {
  // 流事件结束，处理收尾工作
  ...
});
stream.on('error', (err) => {
  // 流事件发生错误，处理错误
  ...
});
```

