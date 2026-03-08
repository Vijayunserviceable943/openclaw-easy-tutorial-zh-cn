---
title: github-openclaw-easy-tutorial
description: github-openclaw-easy-tutorial（最简单的 OpenClaw 入门手册）：从零到接飞书/Telegram/WhatsApp、改人设、多 Agent 实践，零基础可读。
keywords: [OpenClaw, github-openclaw-easy-tutorial, 入门手册, 飞书, Telegram, WhatsApp, AI 助手, 多 Agent, 中文]
---

# github-openclaw-easy-tutorial

**最简单的 OpenClaw 入门手册**（中文）

<!-- 教程元数据 -->
| 项目 | 说明 |
|------|------|
| **名称** | github-openclaw-easy-tutorial |
| **简介** | 最简单的 OpenClaw 入门手册：从零到接飞书/Telegram/WhatsApp、改人设、多 Agent 实践，零基础可读。 |
| **GitHub 简介（可作仓库 About 描述）** | github-openclaw-easy-tutorial — 最简单的 OpenClaw 入门手册，零基础可读。 |
| **语言** | 中文 |
| **适用** | 个人用户、二次开发与多角色协作实践者 |
| **权威参考** | [官方文档](https://docs.openclaw.ai) · [多语言文档 www.openclawx.cloud](https://www.openclawx.cloud) |

---

> 你是不是第一次听说 OpenClaw 或 AI 助手？没关系。这份指南从零开始，用**大白话**带你：先和助手聊上天，再接到飞书、Telegram、WhatsApp，最后玩转多角色协作。不要求你有编程基础；遇到不懂的词（比如 Token、Agent），正文里都会用一句话讲清楚。

---

## 怎么用这份指南？

选一条适合你的路，照着走就行：

- **我就想有个能聊的 AI 助手**  
  从「第一部分」按顺序读下去，做到「接到你常用的 IM」那一步，就可以日常用了。后面的章节需要时再翻。

- **我想自己改配置、接 API 或写小技能**  
  先把第一部分的 1.1～1.4 做完，再打开「第二部分」，一步步来。

- **我想玩「多个助手」或「多角色一起干活」**  
  第一部分跑通后，直接看「第三部分：实践与案例」，照着做就能搭出律师+会计师、产品+开发+测试之类的多角色场景。

- **卡住了、报错了**  
  先看每章末尾的「常见问题」和「延伸阅读」。想要更权威、更完整的答案，可以到 [官方文档](https://docs.openclaw.ai) 或 [多语言文档站 www.openclawx.cloud](https://www.openclawx.cloud) 详细阅读（多语言站对非英文用户更友好）；也可以到项目仓库里搜一搜。

---

## 第一部分：从零到日常 — 先和助手「聊上天」

适合：想用 OpenClaw 当个人 AI 助手、在飞书 / Telegram / WhatsApp 里聊天，暂时不碰代码和复杂配置的你。

| 章 | 标题 | 读完你能做什么 |
|----|------|----------------|
| [1.1](01-getting-started/01-what-is-openclaw.md) | OpenClaw 是什么？我需要准备啥？ | 用一句话搞懂它能干啥； |
| [1.2](01-getting-started/02-first-chat.md) | 十分钟内：安装并在浏览器里和助手说上话 | 一键安装、跟着向导走、在网页里完成第一次对话（不接任何 IM 也能先玩起来） |
| [1.3](01-getting-started/03-connect-im.md) | 让助手出现在飞书 / Telegram / WhatsApp 里 | 选一个你常用的渠道 |
| [1.4](01-getting-started/04-make-it-yours.md) | 让助手更像「你自己」：人设、习惯与记忆 | 找到「工作区」在哪；改哪几个文件能改助手的性格和规矩（AGENTS、SOUL、USER、IDENTITY）；记忆怎么用（可选） |
| [1.5](01-getting-started/05-daily-and-troubleshoot.md) | 日常怎么用、出问题怎么查 | 看日志、重启服务、更新；连不上、收不到消息、权限报错时怎么办；哪里查更详细的文档 |

---

## 第二部分：进阶 — 给助手「定规矩」、接 API、写技能

适合：已经能日常用 OpenClaw 了，想改配置、接 HTTP API、写或安装 Skills、从源码跑或参与贡献的你。

**建议**：先把第一部分的 1.1～1.4 做完，再从这里开始。

| 章 | 标题 | 读完你能做什么 |
|----|------|----------------|
| [2.1](02-advanced/01-config-and-model.md) | 给你的助手定规矩：一分钟搞懂配置文件 | 知道 `openclaw.json` 大致长什么样、怎么改模型和 provider、常用环境变量、多环境/profile 怎么用 |
| [2.2](02-advanced/02-gateway-and-api.md) | Gateway 是啥？怎么用 API 和别的系统对接？ | 搞懂 Gateway 在整条链路里干啥；控制台与远程访问；用 API 发消息、查状态；和第三方（webhook、自动化）怎么接 |
| [2.3](02-advanced/03-agent-and-workspace-deep.md) | Agent 与工作区：会话、队列与沙箱 | 各引导文件怎么被「喂」给助手；会话与队列模式；沙箱、多工作区在什么场景下用、怎么配 |
| [2.4](02-advanced/04-tools-and-skills.md) | 教助手「新本领」：工具与 Skills | 内置工具有哪些、TOOLS.md 干啥用；Skills 从哪加载、怎样写一个简单 Skill；安全与权限注意点 |
| [2.5](02-advanced/05-extend-and-contribute.md) | 从源码运行与参与贡献 | 从源码怎么构建、怎么跑；插件/渠道开发的入口在哪；遇到问题怎么反馈、社区在哪 |

---

## 第三部分：实践与案例 — 一个网关，多个「分身」

适合：已经能日常用 OpenClaw，想玩「多 Agent」「多角色一起干活」、并愿意动手照着做的你。内容基于官方多 Agent、Sub-Agent 文档，可复现。

| 章 | 标题 | 读完你能做什么 |
|----|------|----------------|
| [3.1](03-practice/01-multi-agent-setup.md) | 一个网关，多个「分身」：搭建你的多 Agent 小队 | 搞懂多 Agent 在 OpenClaw 里是啥（独立工作区、会话、渠道账号）；用 `openclaw agents add` 加新助手；用 bindings 把飞书/Telegram/WhatsApp 的消息分给不同助手 |
| [3.2](03-practice/02-subagents-and-tasks.md) | 派助手去「跑腿」：子 Agent 与并行任务 | 子 Agent（Sub-Agent）是啥、啥时候用；用 `/subagents spawn` 派后台任务；用不同模型/角色做并行研究或分工 |
| [3.3](03-practice/03-case-multi-role-legal.md) | 案例：律师 + 会计师 — 多角色聊「诉讼与合规」 | 用多 Agent 分别扮演律师、会计师等，在同一会话或子 Agent 里完成一场「诉讼/合规/财税」讨论；每个角色的人设与配置要点 |
| [3.4](03-practice/04-case-multi-role-dev.md) | 案例：产品 + 开发 + 测试 — 多角色完成一个小项目 | 用多 Agent 扮演产品、开发、测试，协作走完从需求到实现的一小段流程；会话与子 Agent 怎么配合 |

---

## 附录与延伸

需要时再来查即可。

| 条目 | 能帮你什么 |
|------|------------|
| [A. 常见问题速查](appendix/faq.md) | 安装失败、收不到消息、权限错误等集中整理，快速对号入座 |
| [B. 官方文档与资源](appendix/references.md) | 官方文档、多语言站（www.openclawx.cloud）、API、CLI、社区链接，方便深入挖 |
| [C. 术语简表](appendix/glossary.md) | Token、Agent、Gateway、Workspace、Skill 等词一句话解释，扫清阅读障碍 |
| [D. 渠道支持说明](appendix/channels.md) | 飞书、Telegram、WhatsApp 的文档依据；钉钉等以官方渠道列表为准 |

---

## 文档里的小约定

- **加粗**：重要概念或第一次出现的术语，正文里会马上用一句话解释。
- `像这样的文字`：需要你在终端里输入的命令、或实际存在的文件名/路径。
- 每章末尾会有「本节要点」「常见问题」「延伸阅读」，方便你复习和往下深入。

---

*本指南按 OpenClaw 官方文档整理，力求好懂、有用、由浅入深；具体参数与最新行为以 [官方文档](https://docs.openclaw.ai) 或 [多语言文档 www.openclawx.cloud](https://www.openclawx.cloud)（对非英文用户更友好）为准。*
