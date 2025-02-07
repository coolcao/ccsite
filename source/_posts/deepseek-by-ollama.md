---
title: 使用ollama本地部署deepseek大模型
date: 2025-02-07 09:14:56
tags: [deepseek, ollama, AI]
categories:
  - 技术博客
  - 原创
---

deepseek 是最近非常火爆的开源国产 AI 大模型，据说能力不错。前几天在 ollama 模型库里，`deepseek-r1` 模型已上线，我们可以使用 ollama 在本地部署，学习体验一下。

<!-- more -->

## ollama 安装

首先安装 ollama，是一款大模型运行工具。官网是 [https://ollama.com/](https://ollama.com/)。

进入官网，根据不同的平台，下载相应的安装包即可。

这里我使用的是 linux 系统，因此只需要执行 `curl -fsSL https://ollama.com/install.sh | sh` 命令即可安装 ollama。

安装完成后，在终端执行 `ollama --version` 命令查看是否安装成功，如果出现以下版本信息，说明安装成功。

```shell
$ ollama --version
ollama version is 0.5.7
```

## 下载 deepseek 模型

安装完成后，可以在 ollama 官方模型库页面，搜索模型进行安装。

[https://ollama.com/search](https://ollama.com/search)

打开上面链接，输入 deepseek-r1 即可搜索到相应的模型。

![](https://s2.loli.net/2025/02/07/uELMdzZWB84hGeP.png)

点击模型进入模型详情页面。

![](https://s2.loli.net/2025/02/07/d7jZwcDihbP5IWV.png)

其中红色框内可以选择不同参数的模型，参数量越大，模型体积越大，需要的显卡配置就越高，当然效果就会越好。这里根据自己电脑的配置以及显卡配置，选择合适大小的模型。

绿色框内为下载命令，复制到终端执行。

比如我这里以最小的 `deepseek-r1:1.5b` 模型为例，具体的命令为 `ollama run deepseek-r1:1.5b` 。

在此期间，ollama 会自动联网下载相应的模型文件以及配置文件，根据模型大小等待时间不等。下载完成后看到如下界面即启动成功：

```shell
$ ollama run deepseek-r1:1.5b
>>> Send a message (/? for help)
```

这样我们就在终端启动了一个 ollama 交互窗口，可以和 deepseek-r1 模型进行交互了。

## 配置 chatbox

使用终端进行交互，非常不方便，且界面不友好。ollama 本身提供了后台服务 api，可以通过第三方客户端接入 api 的方式进行使用，这里我们使用 chatbox 客户端。

[https://chatboxai.app/zh](https://chatboxai.app/zh)

打开 chatbox 的官网，下载 chatbox 客户端并安装即可。

### 配置 ollama 网络

配置 chatbox 之前，我们需要先配置一下 ollama 的监听网络，默认情况下，其只监听本地网络，如果你想让局域网内的其他电脑也能访问到 ollama 服务，则需要配置其监听网络，如果你只在本地使用，则可以跳过此操作。

编辑 `/etc/systemd/system/ollama.service.d/override.conf` 文件，如果没有，则创建一下即可。

在配置文件的 `[service]` 下添加如下配置：

```conf
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

保存后，执行如下命令重启 ollama：

```shell
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### 配置 chatbox

打开 chatbox 的设置界面，模型提供方选择 ollama api：

![](https://s2.loli.net/2025/02/07/yMKRlnOxW2q4m5V.png)

API 域名这里填 ollama 部署的机器的 IP，端口填 11434 即可，ollama 默认端口。

下面模型选择之前安装并运行的模型即可。

如果配置正确，此时就可以通过 chatbox 和本地部署的 deepseek-r1 模型进行对话了。

![](https://s2.loli.net/2025/02/07/CRSUAbVvlGoYyTa.png)
