---
title: Agent 与工作区：会话、队列与沙箱
description: Bootstrap 文件注入、Session 存储、队列模式 steer/followup/collect、沙箱 workspaceAccess 配置与多 Agent 工作区路径。
keywords: [OpenClaw, Agent, 工作区, Session, 队列, 沙箱, bootstrap]
---

# Agent 与工作区：会话、队列与沙箱

读完这一节，你会搞清：工作区里那几份文件是**怎么被「喂」给助手的**、**会话**存在哪、**队列**几种模式分别啥意思（你发多条消息时助手怎么处理），以及**沙箱**在什么场景下用、怎么配。适合已经改过配置、想细调「谁先跑、怎么跑、跑在哪儿」的你。

---

## 引导文件是怎么被「喂」进助手的

助手每次**新会话**的第一轮，OpenClaw 会把工作区里那几份 **bootstrap 文件**（AGENTS.md、SOUL.md、USER.md、IDENTITY.md、TOOLS.md 等）的**内容**直接塞进助手的上下文里，相当于「一上来就告诉助手：你是谁、规矩是啥、用户是谁」。这就是**注入**（inject）：不是助手自己去读文件，而是 Gateway 在发请求给模型前，先把这些内容拼进提示里。

- **空文件**：会跳过，不注入。
- **文件缺失**：会注入一行「该文件缺失」的标记；`openclaw setup` 可以帮你补建默认模板。
- **文件很大**：会被截断并加标记，避免一次塞太多占满上下文；完整内容仍在磁盘上，助手需要时可以用读文件类工具再读。截断阈值可在配置里用 `agents.defaults.bootstrapMaxChars`、`agents.defaults.bootstrapTotalMaxChars` 调。
- **BOOTSTRAP.md**：只在**全新工作区**（还没有其他引导文件）时自动生成，是一次性的「首次运行仪式」；跑完建议删掉，之后不会再自动建。
- 若你**自己**管理所有引导文件、不想 OpenClaw 自动创建任何一份，可在配置里设 `agent.skipBootstrap: true`，就不会再自动创建这些文件。

子会话（例如子 Agent）一般只注入 AGENTS.md 和 TOOLS.md 等部分文件，主会话才会注入完整一套；细节见官方 [Agent runtime](https://docs.openclaw.ai/concepts/agent)。

---

## 会话：存在哪、怎么区分

一次连续对话从开始到结束，在 OpenClaw 里算一个**会话**（Session）。每个会话有一个稳定 ID，由 OpenClaw 分配。

- **存放位置**：会话记录（对话内容、元数据）以 JSONL 形式存在  
  `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`  
  例如只有一个默认助手时，`agentId` 是 `main`。
- **谁一个会话**：通常「同一个渠道 + 同一个对话对象 + 同一段连续对话」会对应一个会话；具体由 Gateway 的会话路由规则决定（例如按渠道、按发送者、按线程等）。多 Agent 时，每个 Agent 有自己的 `agentId`，会话目录互相独立。

你一般不用直接去改这些 JSONL；只是要知道：**会话 = 一段连续对话的完整记录**，存在上述路径下，重启 Gateway 也不会丢（除非你删掉或换了状态目录）。

---

## 队列模式：你连发多条消息时助手怎么处理

当你在飞书、Telegram 或控制台里**连续发多条消息**时，Gateway 不会同时开多个「助手轮」乱抢，而是通过一个**队列**把请求串起来或按规则合并。具体行为由**队列模式**决定，你可以在配置里设默认，也可以在某些渠道里单独设。

可以简单理解成三种常见模式：

| 模式 | 含义（通俗说） |
|------|----------------|
| **steer** | 新消息**立刻**插进当前这一轮：助手正在干活时收到新消息，会在「下一个合适的位置」停下当前计划，转去处理新消息。适合你希望「发一句就能打断/改方向」的场景。 |
| **followup** | 新消息**不插队**，等当前这一轮完全结束后，再开新的一轮，用新消息当输入。适合不想打断、一条一条排队处理。 |
| **collect** | 多条新消息先**攒着**，等当前轮结束后，把攒着的多条**合并成一条**再交给助手（可配 debounce，避免「继续继续」刷屏）。默认很多渠道用的是 collect。 |

还有 **steer-backlog**（既立刻 steer，又保留一份给下一轮，可能看起来像「回了两次」）、**interrupt**（直接中止当前轮，只处理最新一条）等，详见官方 [Queue](https://docs.openclaw.ai/concepts/queue)。

配置示例（全局默认 + 按渠道覆盖）：

```json
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```

- **debounceMs**：等多少毫秒没有新消息后，再把攒着的消息交给助手，避免你连打几个字就触发多次。
- **cap**：每个会话最多攒多少条；多了就按 **drop** 策略处理（例如丢掉旧的、或合并成一段摘要再注入）。
- 在对话里也可以发 **/queue &lt;mode&gt;** 只改当前会话的队列模式（若渠道支持）。

---

## 沙箱：什么时候用、怎么配

默认情况下，助手的**工具**（读文件、执行命令、写文件等）是直接在你运行 Gateway 的**本机**上执行的。若你希望把「执行」关进一个隔离环境，减少误操作或恶意指令的影响，可以用 **沙箱**（Sandbox）：OpenClaw 用 **Docker 容器**跑工具，让文件、进程都限制在容器里。

- **谁被沙箱**：由 `agents.defaults.sandbox`（以及可选的 `agents.list[].sandbox`）控制。**mode** 决定「哪些会话」进沙箱：
  - `off`：不沙箱，全在本机。
  - `non-main`：只有**非主会话**进沙箱（例如群聊、子 Agent、某些渠道会话），主会话（你私聊、控制台）仍在本机。
  - `all`：所有会话的工具执行都进沙箱。
- **工作区在沙箱里怎么见**：`agents.defaults.sandbox.workspaceAccess`：
  - `none`（默认）：沙箱里看不到你真实工作区，只有一块临时目录（在 `~/.openclaw/sandboxes` 下），适合「完全隔离」。
  - `ro`：把真实工作区以**只读**挂进沙箱（例如 `/agent`），助手能读不能写。
  - `rw`：把真实工作区以**读写**挂进沙箱，助手在沙箱里改的就是你工作区。

沙箱需要本机有 **Docker**，并先按官方文档构建沙箱镜像（如 `openclaw-sandbox:bookworm-slim`）。若你不需要隔离，可以不开沙箱，工具照常在本机跑。

---

## 多工作区：多 Agent 与沙箱里的「每会话工作区」

- **多 Agent**：当你配置了多个 Agent（`agents.list` 里多个 id），每个 Agent 可以有**自己的工作区路径**（在对应 agent 的配置里写 workspace）。这样不同助手用不同目录，互不干扰。详见 [Multi-Agent](/concepts/multi-agent) 和本教程第三部分。
- **沙箱 + 非主会话**：开启沙箱且 mode 为 `non-main` 或 `all` 时，非主会话可以使用**按会话隔离**的沙箱工作区（在 `agents.defaults.sandbox.workspaceRoot` 下），而不是直接用你的主工作区，进一步避免串台。详见官方 [Sandboxing](https://docs.openclaw.ai/gateway/sandboxing)。

---

## 本节要点

- **引导文件**在**新会话第一轮**被注入进助手上下文；空文件跳过、缺失打标记、过大截断；BOOTSTRAP.md 仅全新工作区一次性；可设 `agent.skipBootstrap: true` 禁止自动创建。
- **会话**记录在 `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`，会话 ID 稳定，由 Gateway 分配。
- **队列**：**steer** = 新消息插进当前轮；**followup** = 等当前轮结束再开新轮；**collect** = 攒多条合并再处理；可配 debounce、cap、drop，也可在会话里用 `/queue <mode>`。
- **沙箱**：用 Docker 隔离工具执行；mode 选 off / non-main / all；workspaceAccess 选 none / ro / rw；需 Docker 和沙箱镜像。
- **多工作区**：多 Agent 各配各的 workspace；沙箱下非主会话可用每会话沙箱工作区。

---

## 常见问题

**Q：改了 AGENTS.md 但助手好像没按新规矩来？**  
引导文件只在**新会话的第一轮**注入；若你是在**已有会话**里继续聊，当前会话已经用过旧内容了。新开一个会话（新对话或刷新后重聊）才会读到最新文件。

**Q：队列模式在哪儿设？**  
全局在配置里 `messages.queue.mode` 和 `messages.queue.byChannel.<渠道>`；当前会话也可用 `/queue collect`、`/queue steer` 等（取决于渠道是否支持）。

**Q：开了沙箱后助手说找不到工作区里的文件？**  
若 `workspaceAccess` 是 `none`，沙箱里根本没有挂你的工作区，只有沙箱自己的临时目录。需要读工作区时改为 `ro` 或 `rw`（注意 rw 会让助手在沙箱里改你真实工作区）。

**Q：多 Agent 时每个助手的工作区路径怎么设？**  
在 `agents.list[]` 里给每个 agent 设 `workspace`（或用 agentDir 等）；详见 [Multi-Agent](https://docs.openclaw.ai/concepts/multi-agent) 和本教程 3.1。

---

## 延伸阅读

- Agent 运行时与引导文件（英文）：[Agent runtime](https://docs.openclaw.ai/concepts/agent)  
- 工作区完整布局（英文）：[Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace)  
- 队列模式与选项（英文）：[Queue](https://docs.openclaw.ai/concepts/queue)  
- 沙箱配置与镜像（英文）：[Sandboxing](https://docs.openclaw.ai/gateway/sandboxing)  
- 多 Agent 与工作区（英文）：[Multi-Agent](https://docs.openclaw.ai/concepts/multi-agent)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
