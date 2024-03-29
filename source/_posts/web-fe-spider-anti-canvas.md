---
title: 一种基于Canvas和对称加密的Web前端隐私数据保护方式
date: 2018-08-22 21:39:10
tags: [前端, 爬虫, 反爬]
categories:
- 技术博客
- 原创
---

之前在[爬虫攻防之前端策略简析](https://coolcao.com/2018/06/09/tips-of-anti-spider-in-fe/)这篇文章中简单分析过目前Web前端对于反爬虫而采取的一些方式，比如使用图片，使用自制字体，还有使用CSS错位等方式，这几种方式中比较流行的自定义字体形式，我在文章中也做了一定的分析，处理得当也可以进行破解。

<!--more-->

在Web前端中，想要保护一些关键数据不被爬虫爬取，我们要做的就是增加爬虫的分析难度，爬虫程序越难分析出原始数据，越能保护好企业的核心资产。

> 理论上，只要是通过Web页面渲染出来的数据，爬虫都可以爬取到。我们能做的就是怎样保护不被爬虫拿到结构化数据。比如一些关键性的数据，我们做一些处理，让爬虫程序不那么容易解析出其原始的结构化数据，这种越难解析保护性就越好。


我最近在研究的过程中，发现了一种比自定义字体，图片等方式更优雅的方式：**基于canvas和对称加密**。

具体而言就是，后端接口对于一些关键性的不想被爬虫爬取的数据进行对称加密，然后前端页面拿到数据后，使用相同的密钥进行解密，得到原始数据，再通过canvas的方式渲染到页面，这样不影响页面数据的展示，而对于爬虫程序而言，数据又是加密的，可以很好的保护一些关键性数据。

有了想法，做个实验验证一下。

后端使用koa简单起了一个服务，接口返回用户数据，包括姓名和年龄，其中年龄这个字段是想要保护的数据，对年龄进行加密，api返回的数据如下：

![1](https://img.coolcao.site/file/a69057e7c09c10aa4f1a4.png)


```ts
this.service.getUsers().subscribe(data => {
      if (data) {
        this.user = data as User;
        this.user.age = AES.decrypt(this.user.age, 'test123').toString(enc.Utf8)

        const canvas: HTMLCanvasElement = document.getElementById('age') as HTMLCanvasElement;

        if (canvas) {
          const ctx = canvas.getContext('2d');

          if (ctx) {
            // ctx.scale(window.devicePixelRatio,window.devicePixelRatio);
            ctx.font = "28px serif";
            ctx.fillText(this.user!.age, 0, 30)
          }
        }
      }
    })
```

模板文件：
```html
<div class="content" role="main">
  <div *ngIf="user"><span style="font-size: 28px;">{{user.name}}</span><canvas id="age" width="30" height="30"></canvas></div>
</div>
```

前端拿到数据后，使用相同的密钥和相同的加密方法对年龄字段进行解密，得到原始数据后，动态渲染到canvas上，效果如下：

![2](https://img.coolcao.site/file/3873566dd0e5b54e7877b.png)

在页面上能够正常展示原始的数据，而在dom结构上，年龄这个字段是一个canvas，里面具体数据是动态渲染出来的，dom结构拿不到真实的数据，这样爬虫程序就很难获取到关键的年龄字段，即使是使用无头浏览器也拿不到。

在之前的文章中我已分析过，使用自定义字体的方式，其实是可以破解的，将字体文件下载下来，解析出里面具体文字与编码的对应关系，然后再一一映射即可。

而对于图片这种方式，由于图片分辨率的问题，在显示效果上会差一些，而且如果要调整样式，图片也不是很方便。

而对于使用canvas加对称加密这种方式，canvas的样式定义以及渲染都在前端进行，前端可以很方便的对样式进行调整，而且在显示效果上，几乎和文字显示一致，显示效果更佳。前端的强项就是在于样式，这种方式更具操作性。



