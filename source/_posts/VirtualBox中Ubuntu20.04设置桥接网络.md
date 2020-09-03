---
title: VirtualBox中Ubuntu20.04设置桥接网络.md
date: 2020-09-03 10:25:50
tags: [ubuntu, virtualbox, 虚拟机]
categories:
- 技术博客
- 原创
---


在虚拟机VirtualBox中安装UbuntuServer进行实验，默认使用NAT网络模式，但是宿主机无法联通虚拟机，因此将NAT模式改为桥接模式。

<!-- more -->
## 修改VirtualBox网络模式
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1599100871_20200903103101181_2001375990.png)

改完重启后，如果遇到ubuntu无法联网操作，那么需要手动设置网络。

## 查看宿主机网络

首先我们查看一下宿主机的网络，这里我使用的是Mac，如果你用的其他系统，可能操作和界面不大一样，但我们需要找到如下的信息：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1599100871_20200903103349509_1089900969.png)

1. 宿主机的ip地址，即上面的IPv4地址。
2. 网关地址。即上面的路由器地址

## 设置Ubuntu网络

最新版的ubuntu已不在`/etc/network/interfaces`设置网络，取而代之的是在`/etc/netplan/**.yaml`中设置。

先切换到 `/etc/netplan` 目录下，查看这个目录下的文件：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1599100872_20200903103557158_2077560720.png)
我的这台虚拟机上有个`00-installer-config.yaml`文件，你的可能名字不一致，但应该也是yaml文件。

编辑其中的内容：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1599100872_20200903103836882_1626404467.png)

将dhcp4设置为false，因为我们要指定静态地址。
addresses为要设置的ip地址，这里要设置成和上面宿主机一样的网段的地址。
gateway4为要设置的网关地址，和宿主机网关地址一样。
nameservers为dns地址。

编辑完后保存，然后执行 `sudo netplan apply` 应用设置即可。



