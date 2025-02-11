---
title: 利用ollama搭建本地大模型搭配obsidian搭建个人知识库
date: 2025-02-11 14:58:04
tags: [obsidian, ollama, deepseek]
categories:
  - 技术博客
  - 原创
---

我们在 {% post_link deepseek-by-ollama %} 中已经使用 ollama 搭建了本地的 deepseek，虽然使用个人电脑搭建只能运行小参数的 deepseek 服务，其效果远远不及 deepseek 官网提供的服务那么好，但如果你不是需要一些联网服务以及深度思考的应用，只是用 AI 做一下简单的个人助理，本地搭建的服务也够用。

本文整理一下如何使用 obsidian 和 ollama 搭建的本地大模型搭建个人知识库。

<!-- more -->

## ollama 服务搭建

这部分的内容可以参阅博客 {% post_link deepseek-by-ollama %} 。

## obsidian 插件安装配置

obsidian 需要下载第三方插件，以支持 ollama 服务。
这里推荐两个插件：

- Copilot
  ![](https://s2.loli.net/2025/02/11/6wJQuf12jB9pqrH.png)
- Smart Connections
  ![](https://s2.loli.net/2025/02/11/fXe5n4OWtD3VSo9.png)

这两个插件都可以在 obsidian 中使用 ollama 服务，效果都差不多，这里我以 Copilot 为例。

首先在 obsidian 的第三方插件管理处搜索并安装 Copilot 插件。

安装完成后，启用并进入配置界面。

![](https://s2.loli.net/2025/02/11/Ad8OBhmboK5xUQR.png)

这里，我们先点击 Model 进行模型的配置。

Copilot 插件的需要设置两种类型的模型：聊天模型和嵌入模型。聊天模型就是我们与之对话的大语言模型，嵌入模型是将我们的知识库文档以向量数据的方式进行嵌入存储，提供给大模型做分析处理用。

![](https://s2.loli.net/2025/02/11/8HP6DTKtLIVUvga.png)

我们将上面自带的模型都勾掉不用，自行添加我们的 ollama 模型。

![](https://s2.loli.net/2025/02/11/jcClmWgwzA45XrP.png)

Provider 选择 ollama。Model Name 根据你部署的模型名，自行填写，需要注意的是名称不能差一个字符，可以用命令 `ollama list` 查看已安装部署的全部模型。
Base Url 这里填写 ollama 服务的地址即可。

填写完点击验证，如果没问题，点击添加即可。

然后添加嵌入模型，过程和添加聊天模型一致。

> 嵌入模型也是 ollama 部署的，这里推荐使用 mxbai-embed-large 即可。

添加完模型后，回到 obsidian，在最左侧边栏会添加一个 Copilot 的图标，点击即可打开对话框。

![](https://s2.loli.net/2025/02/11/9rqCam8wcoQ4tpP.png)

如上图所示，我问他在一篇 md 文档中有多少个充电桩，他会将这篇文档中记录的各个属性罗列出来，然后针对问题进行分析，并给出结果。

本地搭建的 deepseek 服务虽然在能力上相比官方服务差不少，但是在这种本地知识库，文档整理与分析这种场景下用，还是可以的，且不会联网，不会将你的文档散发到网络，安全性更高。
所以，如果可以，在本地部署一个 ollama 服务，用于个人知识库，也是一个不错的选择。
