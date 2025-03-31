---
title: MCP服务使用技巧
date: 2025-03-31 09:51:50
tags: [AI, MCP, 大模型, 模型上下文协议]
categories:
  - 技术博客
  - 原创
---

最近 MCP 服务大热，我也是写过两篇关于 MCP 服务的博客 {% post_link mcp %} {% post_link mcp-example %}，来介绍了一下 MCP 的主要概念，以及编写了一个 MCP 服务示例。

在这篇文章中，我将介绍一些 MCP 服务的使用技巧，以及如何使用 MCP 服务来开发一个简单的待办事项应用。

<!-- more -->

## 待办事项的 MCP 服务实现

这里我们使用 MCP 的 nodejs SDK 来实现一个简单的待办事项应用，其主要功能如下：

- addTodo: 添加待办事项
- queryTodos: 查询所有待办事项
- deleteTodo: 删除待办事项
- updateTodo: 更新待办事项
- toggleTodoStatus: 更新待办事项状态

因为涉及到数据持久化，因此我们使用了 lokijs 这个库来实现数据库的操作，简单易用，也支持持久化到磁盘。

项目具体代码地址是：[https://github.com/coolcao/todo-mcp-server](https://github.com/coolcao/todo-mcp-server)

## MCP 服务接入大模型

这里我使用了 CherryStudio 作为 AI 大模型客户端，这个工具在最近的版本中加入了对 MCP 的支持，使用体验尚可。

### 代码下载与编译

1. MCP 服务下载。使用 git 将上面 todo-mcp-server 项目下载到本地。
2. 安装依赖：`npm install`
3. 编译代码：`npm run build`

### 配置 MCP 服务

打开 CherryStudio，点击设置，进入设置页面。
在菜单中找到 `MCP服务器`选项，点击进入 MCP 服务配置。
点击添加服务器，打开添加服务器配置页面：
![202503311009298](https://img.coolcao.site/file/AgACAgUAAyEGAASKxe6JAAMnZ-n5V2HBu9NoSp0ZNxqeTtZugOgAAm7BMRvZCVFXviUVDrf86LcBAAMCAAN3AAM2BA.png)

必填项有如下几个：

1. 名称: 给这个 MCP 服务器起一个名字，方便识别
2. 类型：本地使用的 MCP 服务器，统一选择 `STDIO`
3. 命令：上面这个项目我们使用的是 nodejs 编写的，因此这里填写 `node`
4. 参数：这里填入编译后的 js 文件，`/your/path/to/todo-mcp-server/build/index.js`，这里替换成你自己的路径即可
5. 环境变量：这个参数是可选的，根据 MCP 服务需要，这里我们需要填写一下数据持久化的路径，用来持久化待办事项。`DB=/your/path/to/cherry-studio-todo.db`

配置完成后，点击保存，如果没有问题，会在下面看到当前 MCP 服务的工具列表，如上图中我这里已正确展示了 todo-mcp-server 的工具列表。

### 对话测试

我们新开一个对话，在系统提示词中设置如下：

```
你是一个AI智能助理，主要帮用户处理TODO待办事项，通过工具todo进行待办处理，todo工具有以下几个功能：

- currentDateTime： 获取当前时间
- addTodo: 添加待办事项
- queryTodos: 查询所有待办事项
- deleteTodo: 删除待办事项
- updateTodo: 更新待办事项
- toggleTodoStatus: 更新待办事项状态

你可以通过这几个工具进行待办的管理。

注意以下几点：
1. 待办都涉及到时间，如果你不确定当前时间，可以通过工具来获取，切不可自行猜测时间
2. 待办事项的标题，是唯一的，删除，更新待办等操作需要一字不差的根据标题进行操作
3. 每通过一个接口处理完后，重新整理以下剩余待办给用户
4. 待办事项，默认情况下只需要查询展示未完成的待办即可。除非特别说明，已完成的待办可以不展示。
```

然后我们就可以与大模型进行对话，要求大模型来管理我们的待办事项了。

下图是我的一个测试的对话记录：
![TODO MCP](https://img.coolcao.site/file/AgACAgUAAyEGAASKxe6JAAMoZ-n7PA40eI0x2Asuy3TZWH7Bij0AAnHBMRvZCVFXoa7hVTMIAu8BAAMCAAN3AAM2BA.png)

从截图中可以看出，有了 MCP 服务的加持，大模型可以轻松的帮我们管理待办事项， 这也展示了 MCP 服务的强大功能。

## MCP 服务使用技巧

MCP 服务解决了大模型的信息孤岛问题，使得大模型可以访问到更多的信息，从而提高了大模型的能力。如果我们希望更好的使用 MCP 服务，就需要一些技巧来提高使用效率。

### 编写提示词

有了 MCP 服务，提高了大模型对外部资源的访问，但**MCP 服务并不能提高大模型的基础能力**。对于一些特定的任务，我们需要编写提示词来帮助大模型明确的调用相关工具，来完成具体的任务。
比如上面我们这个 `todo-mcp-server` 这个例子中，我们通过提示词，来明确告诉大模型，要管理待办事项，使用 todo 工具来完成，且在提示词里还要告诉大模型，这个工具的一些注意事项，目的就是让大模型能够准确无误的调用工具，来完成任务。

### 组合调用

大模型在使用 MCP 服务时，并不是只能调用一个工具，而是可以组合调用多个工具来完成任务。
比如我们在上面的例子中，除了 todo 这个服务外，我们还加入了一个`currentDateTime`工具，用来获取当前时间。
这样，我们就可以在提示词中，明确告诉大模型，要获取当前时间，然后再调用 todo 工具来管理待办事项，以确保待办事项的时间是准确的。

### 模型支持

并不是所有的大模型都支持 MCP 服务，只有支持工具调用的大模型才能支持 MCP 服务。
一般情况下，**普通的聊天模型是支持 MCP 服务的**， **而像 deepseek r1 这种推理模型就不支持 MCP 服务。**
这一点需要注意，我们如果要使用 MCP 服务，就需要使用支持工具调用的大模型。

### MCP 服务搜索

MCP 服务可以在 [https://github.com/punkpeye/awesome-mcp-servers/blob/main/README-zh.md](https://github.com/punkpeye/awesome-mcp-servers/blob/main/README-zh.md) 这个项目上找一下有没有自己需要的服务，除此之外，还有一些网站可以搜索，比如：

- [https://mcp.so/zh](https://mcp.so/zh)
- [https://glama.ai/mcp/servers](https://glama.ai/mcp/servers)

当然，如果你是一个开发者，熟悉 nodejs 或者 python ，也可以根据自己的需要，编写自己的 MCP 服务器使用。
