---
title: 解决HTTPS应用访问HTTP图片被禁止的问题
date: 2022-08-10 09:22:55
tags: [http, https, 混合内容]
categories:
- 技术博客
- 原创
---

## 目录
-   [混合内容访问](#混合内容)
-   [打开浏览器对于混合内容的限制](#打开浏览器对于混合内容的限制)
-   [服务器端使用nginx进行转发](#服务器端使用nginx进行转发)

<!-- more -->

## 混合内容

在部署为HTPTS的应用里，有一些HTTP的图片无法正常显示，终端显示的错误如下：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/image_loFXESaSKs.png)

出现这个错误是由`混合内容` 造成的。

> 混合内容是指在HTTPS页面包含HTTP子资源。

`chrome` 浏览器从版本84之后，默认会阻止混合内容，也就造成了今天说的，在HTTPS应用中，HTTP的图片无法正常显示。

`chrome` 浏览器这么做的目的也是为了安全考虑。

对于`混合内容` 的知识，有兴趣的可以参考[火狐开发者文档](https://developer.mozilla.org/zh-TW/docs/Web/Security/Mixed_content "火狐开发者文档")。

那我们应该怎么解决这个问题呢？

***

1.  使用HTTP访问
2.  图片连接改为HTTPS（如果可以）
3.  打开浏览器对于混合内容的限制（客户端）
4.  配置nginx，使用nginx进行转发（服务器端）

对于方法1，不推荐。我们开发应用，肯定要是使用HTTPS部署才有更好的安全性。

从安全性角度考虑，如果图片资源能改为HTTPS的是最好不过了，不过如果由于一些原因，比如使用的CDN只提供了HTTP连接，那么我们可以采用第三种方案，打开浏览器的混合内容的限制。

## 打开浏览器对于混合内容的限制

具体方法如下：

1.  点击浏览器地址栏前的小锁🔒图标，并点击网站设置

    ![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/image_avBDz1io0p.png)
2.  进入网站设置后，下拉找到 `不安全内容` ，选择 `允许`&#x20;

    ![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/image_98CvgMjj9C.png)
3.  然后返回页面，刷新即可正常显示HTTP图片

## 服务器端使用nginx进行转发

对于方法3，仅限于客户端（浏览器端），当我们知道我们的应用采用http混合内容不存在安全问题时，我们可以在浏览器上打开本网站的混合内容的限制。

但如果我们是开发人员，把我们的网站部署在https服务下，当用户访问我们的https网站时，我们无法要求用户知道并理解方法3的内容。此时我们应该在服务器端将http的内容进行转发。

我们将http的内容，加一个前缀标识，然后在nginx上对该标识的内容直接转发到http即可。

比如，我们的http内容连接是cdn连接 [http://cdn-host.com/uploads/file/FpkfOZIViVqYjYBxu0vE-EbBBwMC\_151316.jpg](http://cdn-host.com/uploads/file/FpkfOZIViVqYjYBxu0vE-EbBBwMC_151316.jpg "http://cdn-host.com/uploads/file/FpkfOZIViVqYjYBxu0vE-EbBBwMC_151316.jpg")

我们将其加个前缀，做一下转换 [https://service-host.com/admin/cdn-qiniu/uploads/file/Fikox09Uut1L-tVtwLk6uRoOscuL\_143100.jpg](https://service-host.com/admin/cdn-qiniu/uploads/file/Fikox09Uut1L-tVtwLk6uRoOscuL_143100.jpg "https://service-host.com/admin/cdn-qiniu/uploads/file/Fikox09Uut1L-tVtwLk6uRoOscuL_143100.jpg")

然后在 `service-host` 的nginx做如下转发配置：

```text
location ^~ /admin/cdn-qiniu/ {
    proxy_pass http://cdn-host.com/;
    proxy_set_header   Host             "cdn-host.com";
}
```
