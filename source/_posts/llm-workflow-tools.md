---
title: 推荐几款开源的大模型应用构建平台
date: 2024-04-22 16:08:38
tags: [ai]
categories:
- 技术博客
- 原创
---

随着ChatGPT的火爆，各种大模型也是层出不穷。对于我们作为应用层而言，要想对接那么多的大模型，费时又费力。

今天我就推荐几款开源的大模型应用构建平台，使得对接大模型更加容易，甚至不需要代码基础，基于图形化的工作流便可。

<!-- more -->

## Dify.ai
![dify](https://img.coolcao.site/file/e59958b6c6478811c8bf7.png)
dify是国内比较早的一批AI智能体构建开发平台，已在github开源，可以自行部署。
其官网链接为[官网](https://dify.ai/)
github地址为[github](https://github.com/langgenius/dify)

dify目前已支持4种智能体类型的创建。

![创建](https://img.coolcao.site/file/d1865c381efafb8c40fa1.png)

- 聊天助手：即普通的聊天型机器人的创建。创建后，可以和该机器人进行对话。
  ![聊天](https://img.coolcao.site/file/6a4c04ad4aca05e29e0f2.png)
- 文本生成应用：这是一类特殊的AI机器人，交互界面比聊天机器人简洁，用户只需要一次输入，机器人生成一次输出。
  ![文生生成](https://img.coolcao.site/file/f542835ec5654a8924600.png)
- Agent: Agent可以看作是更复杂的对话模型，利用大语言模型的推理能力，能够自主对复杂的人类任务进行目标规划、任务拆解、工具调用、过程迭代，并在没有人类干预的情况下完成任务。
- 工作流：基于工作流的低代码AI机器人搭建，可以根据业务需要，自主选择对应的节点来搭建业务流程。
  ![workflow](https://img.coolcao.site/file/ea8fa42614286b40454b9.png)

同时，dify还支持工具，以及自定义工具来拓展大模型的能力。
![工具](https://img.coolcao.site/file/dc04b77f8acea3a6998ee.png)

总体而言，dify对于中小企业落地AI大模型应用提供了一个简单易用，可靠的开发平台，十分推荐。

## Flowiseai
flowiseai是一款基于nodejs实现的，图形化大语言模型开发平台，也是开源的。
其官网地址为 [官网](https://flowiseai.com/)
其github地址为 [github](https://github.com/FlowiseAI/Flowise)

![flowiseai](https://img.coolcao.site/file/0d29256ffc0c77a34744e.png)


## Typebot
typebot和flowiseai一样，也是一款开源的，基于nodejs开发的图形化大模型开发平台。
官网地址：[官网](https://typebot.io/)
github地址：[github](https://github.com/baptisteArno/typebot.io)

![typebot](https://img.coolcao.site/file/bd8a735639ecdc198f8a7.png)


## 总结
以上三款都是开源的，github主页都提供了详细的文档以便用户自部署，强烈推荐有需要做AI应用落地的朋友使用。