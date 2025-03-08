---
title: 一文了解MCP协议
date: 2025-03-08 19:36:43
tags: [MCP, AI, 大模型, 模型上下文协议]
categories:
  - 技术博客
  - 原创
---

## 什么是 MCP 协议

MCP(Model Context Protocol)，模型上下文协议，是 Anthropic 公司提出并开源的一项大模型上下文协议标准，专门用来解决大语言模型与各种外部数据来源，工具以及资源之间的交互难题，通过标准化的连接方式增强 AI 应用的实用性和安全性。

现今随着大语言模型的发展，在开发 AI 应用时会遇到许多挑战，如如何获取用户输入、如何处理外部数据等。MCP 协议解决了这些问题，使得大语言模型与各种外部数据来源，工具以及资源之间的交互变得简单易行。它相当于为大语言模型和各种数据源搭建了一座标准化的桥梁，让这些数据源可以轻松地被大语言模型所使用。

<!-- more -->

## MCP 协议的核心架构与组件

MCP 协议基于 **Client-Server**架构，包含五大核心组件：

- MCP Host：发起请求的 LLM 应用，比如 Claude Desktop, AI 驱动的 IDE 等
- MCP Client: MCP 主机内部，与 Server 保持 1:1 连接的客户端，负责协议的协商，工具调用以及数据传输
- MCP Server: 轻量级服务器，通过标准化的接口暴露本地或远程资源（如数据库，文件，API 等），提供工具，资源和提示词三类功能
- Local Resources: 本地资源，本地计算机中可供 MCP 服务器安全访问的资源，例如文件，数据库
- Remote Resources: 远程资源，通过网络连接可访问的远程资源，例如云服务

![202503082034962](https://img.coolcao.site/file/AgACAgUAAyEGAASKxe6JAAMiZ8w5VvZpzWoUED1siX8oFQwsaygAArjCMRtMUGhW4wH3rdoW1qcBAAMCAAN4AAM2BA.png)

### MCP Client

MCP Client 充当 LLM 和 MCP Server 之间的桥梁，工作流程如下：

- MCP Client 首先从 MCP Server 获取可用的工具列表。
- 将用户的查询连同工具描述一起发送给 LLM
- LLM 决定是否需要使用工具，以及使用哪些工具
- 如果需要使用工具，MCP Client 会通过 MCP Server 执行相应的工具调用
- 工具调用的结果会被发送回 LLM
- LLM 基于所有的信息生成自然语言响应
- 最后将响应返回给用户

### MCP Server

MCP Server 是 MCP 架构中的关键组件，它提供 3 种主要类型的功能：

- 资源：类似文件的数据，可以被客户端读取
- 工具： 可以被 LLM 调用的函数
- 提示： 预先编写的模板，帮助用户完成特定的任务

这些功能使 MCP Server 能够提供丰富的上下文信息和操作能力，从而增强 LLM 的实用性和灵活性。
MCP Server 仅暴露受控功能，敏感操作需要用户确认，且默认部署在本地，避免远程攻击。

### 通信机制

MCP 定义了 Client 和 Server 进行通讯的协议与消息格式，其支持两种类型的通讯机制：标准输入输出，基于 SSE 的 HTTP 通讯，分别对应着本地与远程通讯。

Client 和 Server 间使用 JSON-RPC2.0 格式进行消息传输。

本地通讯：使用了 stdio 传输数据，具体流程 Client 启动 Server 程序作为子进程，其消息通讯是通过 stdin/stdout 进行的，格式为 JSON-RPC 2.0。
远程通讯：Client 与 Server 可以部署在任何地方，Client 使用 SSE 与 Server 进行通讯，消息的格式为 JSON-RPC 2.0，Server 定义了 SSE 与 messages 接口用于推送与接收数据。

## 技术特点与优势

1. 模块化设计：通过添加不同 Server 拓展功能（类似插件），例如 GitHub Server 可以与 GitHub 进行数据交互，SQLite Server 可以与 SQLite 进行数据交互。
2. 协议开放性：支持社区贡献 Server 实现，避免依赖中心化平台。
3. 与 Function Calling 的互补：

- MCP：适合复杂的上下文管理（例如多步骤的数据库操作）
- Function Calling：适用于明确的功能调用场景（例如单次的 API 请求）
  |类别|MCP|Function Calling|
  |---|---|---|
  |性质|协议|功能|
  |范围|通用（多数据源，多功能）|特定场景（单一数据源或功能）|
  |目标|统一接口，实现互操作|拓展模型能力|
  |实现|基于标准协议|依赖于特定模型实现|
  |开发复杂度|低，通过统一协议实现多源兼容|高，需要为每个任务单独开发|
  |复用性|高，一次开发，多场景使用|低，函数通常为特定任务设计|
  |灵活性|高，支持动态适配和拓展|低，功能拓展需要额外开发|
  |常见场景|复杂场景，如跨平台数据访问与整合|简单任务，如天气查询|

4. 数据安全性：MCP 通过本地服务器运行，避免将敏感数据上传到第三方平台，确保数据隐私。

## MCP 使用样例

目前支持 MCP 的大模型应用主要有 `claude desktop`， 编程工具 `cursor`， `windsurf`，以及 vscode 插件 `cline` 等，这里我们使用 `windsurf` 来演示一个 MCP 服务器的使用样例。

> 很遗憾，MCP 协议是 Anthropic 公司提出的协议，由于还很新，OpenAI 目前对其还没有跟进，希望后续 OpenAI 也能支持跟进，在 ChatGPT 客户端加入 MCP 协议的支持。
>
> 目前已经有很多开源的三方的 MCP 服务器，你可以在这个 GitHub 仓库里查找自己感兴趣的项目：
> [https://github.com/punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)
>
> 同时，如果你不使用如上这几个工具，你可以从下面这个 git 仓库中找到适合你的 MCP 客户端：
>
> [https://github.com/punkpeye/awesome-mcp-clients/](https://github.com/punkpeye/awesome-mcp-clients/)

本示例中，我将使用 `windsurf` 作为客户端，使用 `mcp-server-sqlite` 作为 MCP 服务器，将 sqlite 的相关操作通过 MCP 协议对接到大模型，做一个简单的待办事项的示例。

MCP 服务器的配置就是一个简单的 JSON 文件，结构如下：

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "uv",
      "args": [
        "--directory",
        "/Users/coolcao/code/mcp-servers/src/sqlite",
        "run",
        "mcp-server-sqlite",
        "--db-path",
        "/Users/coolcao/code/测试sqlite.db"
      ]
    }
  }
}
```

配置好 windsurf 的 sqlite mcp 服务器后，我们就可以直接用 windsurf 的大模型对话，通过大模型来直接调用 sqlite 相关操作了。
这里我将其作为一个智能 AI 助手，帮我记录待办事项，具体的对话如下：

![202503082324673](https://img.coolcao.site/file/AgACAgUAAyEGAASKxe6JAAMjZ8xhVEZtSjr9wTOu54MzOkrXYnYAAk_CMRt4XmlWowLehnyuuBUBAAMCAAN3AAM2BA.png)

我们可以看到，在跟大模型对话的过程中，大模型会自动调用已经配置的 MCP 服务器，并通过相应的操作来直接操作 SQLite 数据库。
我们要大模型给我们做待办事项的管理，对于数据库的任何操作，都是大模型自行调用 sqlite 的 MCP 服务器完成的，创建表，插入数据，查询数据，修改数据等操作，可谓是一气呵成。

而且在整个对话的过程中，对话记录中能够显示大模型在调用 MCP 操作的具体的方法与参数，以便于即使大模型出错了，我们也可以及时通过记录来定位问题，并对其进行修正。

我上面这个简单的例子，最终落到 sqlite 数据库中，数据如下：
![202503082329912](https://img.coolcao.site/file/AgACAgUAAyEGAASKxe6JAAMkZ8xiVuOX-bRfLnJ78tTOZjn2-bYAAmDCMRt4XmlWlNWou45dNEEBAAMCAAN5AAM2BA.png)

通过这个例子，我们可以看出，由于 MCP 服务器的存在，大模型在处理外部数据时更加的便捷高效。
在使用 MCP 服务器之前，我们可能只是要大模型生成对应的 SQL 语句，然后手动复制 SQL 语句到数据库中，然后执行查询。
但是有了 MCP 服务器之后，大模型就可以主动获取数据，并且自动将数据插入到数据库中，大大简化了大模型处理外部数据资源的便利性。

## 总结

MCP 协议的出现，就像给 AI 助手开辟了一条全新的道路，让它们能够主动的获取信息，轻松与各种外部工具进行交互，这意味着未来的工作场景将会更加融合 AI，人与 AI 的协作模式也将更加的高效。

最后，希望 OpenAI 能够早日跟进 MCP 协议吧。
