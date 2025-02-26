---
title: Memobase：为你的AI应用注入长记忆
date: 2025-02-26 10:01:29
tags: [AI, 长记忆, memobase]
categories:
  - 技术博客
  - 原创
---

随着 AI 的发展，AI 应用也越来越火爆，但随之而来也会有一些挑战，比如 AI 记忆。

<!-- more -->

## AI 记忆

为什么 AI 应用需要记忆？
所谓的 AI 记忆是指其存储和调用过往信息的能力，用于提升后续决策的连贯性和相关性。

AI 记忆的类型：

- 短期记忆（上下文窗口）
  在单次交互中（如聊天对话），AI 通过固定长度的上下文窗口记住最近的输入。例如，GPT-4 可能保留约 8000 字的对话历史，但超出部分会被截断。
- 长期记忆
  通过外部数据库（如向量数据库）存储结构化信息，使 AI 能长期访问特定数据。例如，客服机器人调用用户历史订单记录。
- 隐性知识记忆
  预训练模型（如 BERT、GPT）通过海量数据训练获得的通用知识，类似人类的常识储备。

由于 AI 上下文长度的限制，一般 AI 只能记住窗口上下文内的东西，也即我们上面说的短期记忆。
如果要实现长期记忆，需要借助外部数据库来存储，并通过数据库的访问调用，来为 AI 应用注入长记忆功能。

## memobase: AI 应用长记忆解决方案

memobase 的官方地址： [https://www.memobase.io/](https://www.memobase.io/)
github 地址：[https://github.com/memodb-io/memobase](https://github.com/memodb-io/memobase)

memobase 是一个基于用户信息的开源 AI 长记忆解决方案，它有如下特点：

- 基于 UserProfile 架构
  - 结构化存储，易于理解和检索： Memobase 围绕用户 profile 构建记忆，提供清晰的结构，便于存储和调用信息。
  - 量身定制： 你可以根据自己产品需要自定义 profile 结构。例如，健身应用可以存储用户的锻炼历史、饮食偏好和健身目标，而语言学习应用则可以记录词汇进度、语法熟练度和学习偏好。
  - 随用户成长动态调整: 随着 AI 对用户了解的深入，profile 内容可以动态更新和扩展，提供更个性化的体验。
- 隐私与安全
  - 数据所有权： Memobase 让你完全掌控用户的数据。你来决定存储什么、怎么用以及保留多久。
  - 隐私至上： 内置的隐私保护功能确保符合 GDPR 等数据隐私法规。
  - 审计与监控： 提供了工具来监控记忆使用情况、跟踪变化并确保数据完整性。
- 可扩展性与高性能
  - 可扩展性： Memobase 可以轻松支持数百万用户，并随你的应用扩展而无缝扩展。
  - 数据库级的效率： Memobase 借鉴成熟的数据库设计原则，确保快速、高效的信息存储和检索。
  - 最低延迟： 经过性能优化，确保 AI 能在毫秒级别访问所需信息。
- 开发者友好
  - 简单易用的 API： Memobase 提供干净、直观的 API，方便与现有工作流集成。
  - 内置批量处理： 不用你自己构建异步更新机制，Memobase 已为你打包好所有必要功能。
  - 丰富的文档与支持： 我们提供详细的文档和支持社区，帮你快速上手。
- 内置记忆模板
  - 利用最佳实践： Memobase 的模板融入了来自教育、虚拟伴侣、游戏等领域的最佳实践，为主流 AI 产品品类预设了一系列高效的记忆模板。
  - 快速开发，专注创新： 通过直接使用模板，你可以节省时间，将精力集中在打造应用的核心差异化功能上。
  - 示例： 比如，AI 伴侣模板可能会包含预定义的字段，像个性特征、用户兴趣（书籍、电影、音乐、爱好）、关系历史（重要日期、共享经历）、沟通风格偏好，甚至还有内部笑话。它还可以建议根据对话更新记忆的方法，比如情感分析来衡量用户的情绪状态，主题提取来识别关键讨论点。你可以直接使用这些模板，或者根据自己的特定需求进一步定制它们。

memobase 代码分为服务端和客户端。
服务端为核心，采用 python 编写，实现了对接 llm 大模型进行用户信息提取以及对其持久化，还有对外提供了一个 rest api 接口。
客户端代码有 python 的实现，go 的实现以及 typescript 的实现，方便不同语言的项目进行对接。

> 即使你采用的不是 python, go, typescript，也可以直接通过其后端提供的 rest api 进行交互。

### 后端服务搭建

memobase 后端，采用 python 编写，外部依赖 pg 数据库和 redis。

可以直接使用 docker-compose 一个脚本进行搭建，也可以直接使用 docker 镜像，对接已有的 pg 数据库和 redis。

> 搭建过程还是很简单的，但需要解决网络问题，否则有可能因为网络导致安装依赖失败。

1. 使用 docker-compose 一键启动

   - 安装 docker-compose
   - 复制配置文件并根据需要修改

   ```
   cd src/server
   cp .env.example .env
   cp ./api/config.yaml.example ./api/config.yaml
   ```

   > .env 文件为环境配置文件，里面配置了服务端口，数据库的配置，redis 的配置等
   > ./api/config.yaml 为后端服务 api 的配置，里面重要的需要配置 `llm_api_key`和`llm_base_url`即对接的大模型的 key 和 baseUrl

   - `docker-compose build && docker-compose up`命令启动

2. 单独启动 docker 镜像，对接已有服务
   - 如果你已经有 pg 数据库和 redis 数据库，可以直接使用 docker 镜像，然后对接上 pg 数据库和 redis 即可。
   - 拉取镜像 `docker pull ghcr.io/memodb-io/memobase:latest`
   - 配置 config.yarml 和 env.list 文件
   - `docker run --env-file env.list ./api/config.yaml:/app/config.yaml -p 8019:8000 ghcr.io/memodb-io/memobase:main`启动镜像

### 客户端服务测试

客户端官方已经实现了 python, golang, typescript 语言的客户端，这里我们看一下 typescript 的样例代码：

```ts
import { MemoBaseClient, Blob, BlobType } from "@memobase/memobase";
// 初始化客户端
const client = new MemoBaseClient(
  process.env["MEMOBASE_PROJECT_URL"],
  process.env["MEMOBASE_API_KEY"]
);

const main = async () => {
  // 检查后端服务是否联通
  const ping = await client.ping();
  console.log(ping);

  // 添加用户，这里用户id为uuid
  let userId = await client.addUser();
  console.log(userId);
  // 更新用户的额外信息
  userId = await client.updateUser(userId, { name: "John Doe" });
  console.log("Updated user id: ", userId);
  // 获取用户
  let user = await client.getUser(userId);
  console.log(user);
  // 向用户插入对话文档
  const blobId = await user.insert(
    Blob.parse({
      type: BlobType.Enum.chat,
      messages: [
        {
          role: "user",
          content: "Hello, how are you?",
        },
      ],
    })
  );
  console.log(blobId);
  // 获取用户文档信息
  const blob = await user.get(blobId);
  console.log(blob);
  // 调用flush解析并持久化用户记忆
  const flushSuc = await user.flush(BlobType.Enum.chat);
  console.log("Flush success: ", flushSuc);

  const blobs = await user.getAll(BlobType.Enum.chat);
  console.log(blobs);

  user = await client.getOrCreateUser(userId);
  console.log(user);
  // 获取用户的记忆信息
  const profiles = await user.profile();
  console.log(profiles);

  if (profiles.length > 0) {
    const isDel = await user.deleteProfile(profiles[0].topic);
    console.log("Delete profile success: ", isDel);
  }

  const isDel = await client.deleteUser(userId);
  console.log(isDel);
};

main();
```

### 记忆标签自定义

memobase 本身内置了一些记忆标签，英文的标签，分两级，大致如下格式：

```
interest:toys:user喜欢玩的玩具包括小兔子、小乌龟、小马宝莉、小狗、小跳舞、小燕、独角兽、小猫咪
basic_info:age:user的年龄未明确提及，但提到玩具，可能暗示用户是儿童或青少年
contact_info:weather:user提到天气很冷
interest:animation:user's favorite animation is 米小圈上学记
basic_info:name:用户希望被称为西西
interest:favorite_name:user likes the name "独角兽" and "小白"
interest:nickname:user's nickname is 小粉兔
```

在业务场景中，如果你需要中文的记忆标签，那么，可以在 api/config.yaml 文件中使用`overwrite_user_profiles`重写：

```yaml
overwrite_user_profiles:
  - topic: "基本信息"
    sub_topics:
      - "生日"
      - "职业"
      - "性别"
      - "年龄"
      - "别名"
      - "地点"
  - topic: "兴趣爱好"
    sub_topics:
      - "娱乐与休闲"
      - "兴趣领域"
      - "喜欢的话题"
  - topic: "衣食住行"
    sub_topics:
      - "喜欢的食物"
      - "忌口"
      - "饮食习惯"
      - "喜欢的衣服"
      - "妆容打扮"
      - "交通工具"
  - topic: "话题"
    sub_topics:
      - "喜欢的话题"
      - "语气偏好"
      - "沟通方式"
      - "语言"
```

这样大模型就会根据自定义的 topic 和 sub_topics 作为自定义的记忆标签，随即产出的记忆数据如下：

```
兴趣爱好:喜欢的话题:用户喜欢小鸟，对小鸟有强烈的兴趣，喜欢听故事，包括猪大战的故事、萌鸡小队的小魔之战的故事和小蝌蚪找妈妈的故事。用户最喜欢的动画片是米小圈上学记，并且对春天的古诗感兴趣。
兴趣爱好:娱乐与休闲:用户希望和助手一起聊天和玩游戏，喜欢娃娃，拥有二十个娃娃，最喜欢的小天鹅娃娃和独角兽，并且对故事和记忆书感兴趣。用户喜欢玩玩具，包括小兔子、小乌龟、小马宝莉、小狗、小燕、小独角兽和小猫咪。用户想要玩抓人游戏和真心话大冒险，还希望听一个睡前小故事。
基本信息:家庭关系:用户提到父亲，暗示用户与家人有联系，包括爸爸、外婆、爷爷和奶奶。
基本信息:地点:用户知道广州，并询问关于广州和北京的信息，且提到天气情况，暗示用户可能在一个寒冷的地方。
衣食住行:喜欢的食物:用户提到喜欢吃屎
交通工具:交通需求:用户询问从广州飞往北京的飞机
话题:沟通方式:用户以提问的方式与助手交流
基本信息:别名:用户希望被叫西西
兴趣爱好:兴趣领域:用户对娃娃的种类和特点表现出浓厚的兴趣。
```
