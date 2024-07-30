---
title: Obsidian安卓端如何与PC端项目进行同步
date: 2024-03-30 14:07:01
tags: [obsidian, termux]
categories:
- 技术博客
- 原创
---

Obsidian 是一款非常好的**离线**笔记软件，支持Markdown，还有非常多的三方插件来拓展功能，是我目前用的最多的笔记软件。
但移动端和PC端的数据同步，如果不买官方的同步服务，还是有点麻烦的。

<!-- more -->

我对于笔记项目的管理，由于是markdown轻量的纯文本笔记，都是使用git来管理。git直接将项目同步到github仓库。

移动端（我主要使用Android设备）没有合适的git工具，使用git进行同步笔记仓库还是比较麻烦的。
经过探索，摸索出一个不错的道路，比较适合程序员思维，下面就分享给大家。

具体的思路就是，手机上安装termux（安卓系统下一个Linux命令行模拟器，能够安装一些常用的linux命令），然后安装git，在termux上使用git进行obsidian仓库的同步。

下面是具体的步骤与说明。

## 安装Termux
首先需要在安卓手机上安装termux软件，官网在[这里](https://termux.dev/cn/index.html)，有两个渠道可以下载安装包，github和F-Droid，这里我直接从github下载安装。

![](https://img.coolcao.site/file/07236121504f5764056b2.png)

> github上下载安装包，从项目的releases下载最新版的安装包即可。

## 配置Termux
termux安装完后，需要更新配置，挂载手机磁盘，安装需要的软件等等。

### 更新系统
1. 使用 `termux-change-repo` 命令切换源。选择一个国内的源，会加速系统升级，软件安装的速度
![](https://img.coolcao.site/file/9e3beb963b484c407e43d.jpg)

![](https://img.coolcao.site/file/d511cdd78ae43a0e2e635.jpg)
这里选第一个，使用镜像组。

![](https://img.coolcao.site/file/fa094fa49958a7993aa14.jpg)
这里选第三个，中国的镜像源

2. `pkg upgrade` 命令更新源
![](https://img.coolcao.site/file/5b8ce5b3c6959c840486d.jpg)
在更新时，会遇到软件包是否要更新的提示，这里输入y，更新即可。

3. `pkg update` 更新系统

### 安装软件
1. 安装git： `pkg install git`
![](https://img.coolcao.site/file/bb27a9950dbf41fd4ab07.jpg)
2. 安装openssh: `pkg install openssh`
3. 安装vim： `pkg install vim`

### 挂载手机磁盘
使用命令`termux-setup-storage`挂载手机存储到termux下，这样termux就可以访问手机的磁盘空间了。
![](https://img.coolcao.site/file/497935a4b0e4c18ae13b5.jpg)
确认是否要挂载手机磁盘，这里输入y，挂载即可。

![](https://img.coolcao.site/file/9b1c273b0c31199cb05d8.jpg)
挂载完成后，可以输入ls命令来查看是否已挂载成功，如图，如果能看到 `storage` 目录，说明手机磁盘已挂载成功。

手机上的文件，都将在 storage 目录下。

## 同步obsidian仓库
进入 storage 目录下的 shared 目录。

然后在 shared 目录下创建一个新的目录，用于存放obsidian仓库文件。这里我为了在手机上方便查找，使用 `0000` 作为目录名，创建完成后，进入该目录。

然后就可以使用 git 命令来克隆已push到github的obsidian项目了。克隆完毕后，可以在移动端的obsidian上直接打开项目即可。




