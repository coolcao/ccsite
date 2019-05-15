---
title: 前端项目如何使用sonar qube进行代码质量检查
date: 2019-05-15 20:27:59
tags: [sonar, 代码质量检查]
categories:
- 技术博客
- 原创
---


在做Java项目的时候，我们经常会使用 Sonar Qube来进行代码质量检查工作。查看了一下其文档，sonar qube不仅可以做Java的检查，还支持其他语言，比如js, ts等等。

本文简单记录如何配置sonar服务，如何使用其进行前端项目的代码质量检查工作。

<!-- more -->

## 有eslint, tslint等工具，还要sonar干嘛
首先需要说的是，这两者不是一个层级的东西，eslint, tslint是js代码，ts代码的风格检查工具，其定义一些代码编写风格，主要通过这些风格规范个人的代码。

而sonar是一个代码质量管理平台，其支持多种语言，多种检查工具，并将这些工具的结果统一化展示，比如对于js,ts代码，sonar就有eslint，tslint等的插件可以集成进去，统一检查。

## Sonar环境配置
### 下载安装Sonar
安装Sonar有两种方式啊，一种是直接安装二进制的包，然后再配置。第二种就是使用docker，直接pull仓库里的sonar镜像，然后启动一个容器服务即可。

这里我使用docker这种形式，方便快捷。


```
// 启动一个mysql5.7的服务
docker run -p 3307:3306 --name sonardb -e MYSQL_PASSWS=your_passwd -d mysql:5.7

// 下载镜像
docker pull sonarqube

// 启动服务
docker run -d --name sonarqube \
    --link sonardb:sonardb \
    -p 9000:9000 \
    -e sonar.jdbc.username=root \
    -e sonar.jdbc.password=your_passwd \
    -e sonar.jdbc.url="jdbc:mysql://sonardb:3306/sonar?useUnicode=true&characterEncoding=utf8&useSSL=false" \
    sonarqube
```

这个命令会启动一个sonarqube服务，并绑定到端口9000.设置mysql 数据库的参数，username，passwd以及url。

*注意：这里使用的是link方式连接数据库，因为我的mysql也是容器启动的。这样的好处是，即使两个容器重新，ip变了，这里连接也不会发生变化。具体可以参考docker中 --link 的相关知识。*

> * 注意：数据库要使用5.6+，但是目前好像还不兼容8.0版本以上的，好像8.0以上的版本语法有区别，目前在sonar启动创建表结构时，会报错。我这里用的是mysql5.7
> * 第一次启动时，非常慢，因为要初始化表结构以及数据，要耐心等待。。。

### 项目配置
打开localhost:9000，直接登录。默认用户名和密码都是admin。登录完成后，修改密码。

1. 创建项目：

2. 输入key和name，这里的key不是密钥的意思，是项目的唯一标识，一般情况下，key和name都用项目的名称即可。

  ![](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Jietu20190515-210124.png)

3. 生成token，生成token时会要求输入一个密钥，如果不输入的话会直接使用项目名，安全起见，输入一个随机字符串。

  ![](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Jietu20190515-210526.png)

4. 选择语言，进行构建即可。因为是js,ts项目，所以需要额外下载sonar-scanner，安装完成后，直接使用下面给的命令进行检查即可。其中参数是上面步骤设置过的。

  ![](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Jietu20190515-210636.png)

检查一下，看一下结果如何：

![](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Jietu20190515-210927.png)

代码检查是通过了，但是提示有一个bug的警告：

![](https://img001-10042971.cos.ap-shanghai.myqcloud.com/blog/Jietu20190515-211053.png)

这个文件是我重写的一个 BrowserRouter 的工具，明显这个if判断，如果没传basename的话直接返回location，但是这里遗漏了return语句。加上return即可。

虽然我之前配置了tslint来检查代码风格，但是还是遗漏了这里这个潜藏的bug。

所以即便是前端项目代码，也应该使用sonar这种工具来进行代码质量的检查，发现更多潜藏的bug，上线时遇到的问题可能就更少。

其实sonar里面还有好多功能，比如单元测试，覆盖检测等等。更多有趣功能，慢慢发现。