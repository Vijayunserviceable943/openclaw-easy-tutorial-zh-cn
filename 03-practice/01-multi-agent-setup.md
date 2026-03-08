---
title: 一个网关，多个「分身」：搭建你的多 Agent 小队
description: OpenClaw 多 Agent 概念、openclaw agents add、bindings 按渠道/账号/发送者路由消息，飞书 Telegram WhatsApp 示例。
keywords: [OpenClaw, 多 Agent, bindings, agents add, 飞书, Telegram, WhatsApp]
---

# 一个网关，多个「分身」：搭建你的多 Agent 小队

读完这一节，你会搞懂**多 Agent** 在 OpenClaw 里是什么意思（一个助手 = 一套工作区 + 一套会话 + 可选的独立渠道账号），会用 **`openclaw agents add`** 加新助手，并用 **bindings** 把飞书、Telegram、WhatsApp 等渠道的消息**分给不同的助手**。一个 Gateway 进程里可以同时跑多个「分身」，互不串台。

---

## 「一个 Agent」到底是什么

在 OpenClaw 里，一个 **Agent**（助手）就是一个**完整独立的脑子**，拥有：

- **自己的工作区**（Workspace）：里面是 AGENTS.md、SOUL.md、USER.md、记忆等；助手读写文件时默认就在这个目录。
- **自己的状态目录**（agentDir）：存这个助手的认证、模型配置等，路径一般在 `~/.openclaw/agents/<agentId>/agent`。
- **自己的会话库**：对话记录在 `~/.openclaw/agents/<agentId>/sessions/` 下，和别的助手完全分开。

也就是说：**多 Agent = 多套工作区 + 多套会话 + 多套认证**，彼此隔离。同一个 Gateway 可以同时服务多个这样的 Agent；**谁处理哪条消息**，由 **bindings**（绑定规则）决定。

---

## 默认：只有一个 Agent（main）

不配多 Agent 时，OpenClaw 只跑一个助手：

- **agentId** 固定为 **main**。
- 工作区默认 `~/.openclaw/workspace`，会话在 `~/.openclaw/agents/main/sessions/`。
- 所有渠道来的消息都会交给这个 main 助手。

下面说的「多 Agent」就是在这一层之上，再多加几套「脑子」，并靠 bindings 把不同渠道/不同账号/不同人的消息分给不同的 agentId。

---

## 搭建多 Agent 的四个步骤

### 第一步：用向导加新 Agent

在终端执行：

```bash
openclaw agents add work
```

把 `work` 换成你想起的名字（如 `coding`、`social`）。执行后，OpenClaw 会：

- 在配置的 **agents.list** 里加上这个 Agent；
- 为它建一个**独立工作区**（默认如 `~/.openclaw/workspace-work`），并放入 SOUL.md、AGENTS.md 等初始文件；
- 为它建好 **agentDir** 和会话目录。

再加一个就再执行一次，例如：

```bash
openclaw agents add coding
openclaw agents add social
```

每个 Agent 的 **id** 就是你在 `add` 时写的名字（如 `work`、`coding`）；后面写 bindings 时用这个 id 指定「谁处理哪类消息」。

### 第二步：为每个 Agent 准备渠道账号（若要用不同号/不同机器人）

若你希望「工作助手」和「生活助手」用**不同的**飞书应用、不同的 Telegram 机器人或不同的 WhatsApp 号，就需要在对应平台各建一套，并在 OpenClaw 里用 **accountId** 区分。

- **飞书**：在开放平台再建一个应用，或沿用同一应用；在 OpenClaw 配置里用 `channels.feishu.accounts.<accountId>` 区分（若有多个应用）。
- **Telegram**：用 BotFather 再建一个机器人，拿新 Token；在配置里写到 `channels.telegram.accounts.<accountId>.botToken`，每个 accountId 对应一个 Agent 时，后面用 bindings 把该 account 绑到对应 agentId。
- **WhatsApp**：每个号码要单独扫码登录，例如：
  ```bash
  openclaw channels login --channel whatsapp --account personal
  openclaw channels login --channel whatsapp --account work
  ```
  配置里会有 `channels.whatsapp.accounts.personal`、`channels.whatsapp.accounts.work` 等；再通过 bindings 把 `accountId: "work"` 绑到 Agent `work`。

若你暂时只想「一个 Telegram 号、一个 WhatsApp 号」给多个 Agent 用，也可以：用**同一个** channel 账号，靠 **bindings** 按**发送者**（peer）把不同人的私聊分给不同 Agent（见官方「One WhatsApp number, multiple people」示例）。那样的话就不需要多个账号，只需多个 workspace + 多条 binding 规则。

### 第三步：在配置里写好 bindings，把消息路由到对应 Agent

**Bindings** 就是「谁的消息 → 交给哪个 Agent」的规则。在 `~/.openclaw/openclaw.json` 里会有 **bindings** 数组，每条长这样：

```json
{
  "agentId": "work",
  "match": { "channel": "telegram", "accountId": "work" }
}
```

表示：**来自 Telegram、且账号是 work 的那台机器人**的入站消息，都交给 **work** 这个 Agent。  
若你只有一个 Telegram 账号，可以按**发送者**（peer）分，例如：

```json
{
  "bindings": [
    { "agentId": "main", "match": { "channel": "telegram", "peer": { "kind": "direct", "id": "tg:你的个人ID" } } },
    { "agentId": "work", "match": { "channel": "telegram", "peer": { "kind": "direct", "id": "tg:同事ID" } } },
  ]
}
```

规则是**最具体的那条优先**：先看是否匹配某条 binding（channel、accountId、peer 等），匹配到就用对应的 agentId；都匹配不到就用**默认 Agent**（在 agents.list 里标了 `default: true` 的，或第一个）。

飞书、WhatsApp 同理：在 **match** 里写 `channel: "feishu"` 或 `channel: "whatsapp"`，再按需加上 **accountId** 或 **peer**。完整字段和示例见官方 [Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)。

你也可以用 CLI 加 binding，不必手改 JSON：

```bash
openclaw agents bind --agent work --bind telegram:work
openclaw agents bindings
```

`openclaw agents bindings` 会列出当前所有绑定，便于核对。

### 第四步：重启 Gateway 并验证

改完配置后重启 Gateway，再确认路由和渠道都正常：

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

- **agents list --bindings**：看每个 Agent 绑了哪些 channel/account。
- **channels status --probe**：看各渠道是否已连接。  
然后分别在对应渠道（或对应账号）发消息，看是否进到了你期望的那个助手。

---

## 多 Agent 能用来干啥（心里有个数）

- **多人共用一台 Gateway**：每人一个 Agent，各自的工作区、会话、人设互不干扰；用 bindings 把「你的飞书/Telegram/WhatsApp」指向你的 Agent。
- **一个号多个「角色」**：同一个 Telegram 或 WhatsApp 号下，按**私聊对象**（peer）把不同人的对话分给不同 Agent（例如客服 Agent、个人助理 Agent）。
- **多角色协作**：下一节会讲**子 Agent**（Sub-Agent）；主助手可以在一次对话里派「子助手」去跑任务，适合多角色讨论、分工协作等。

---

## 本节要点

- **一个 Agent** = 一套工作区 + agentDir + 会话库，彼此隔离；多 Agent 就是在一个 Gateway 里跑多套这样的「脑子」。
- **加新 Agent**：`openclaw agents add <名字>`，会自动加进 agents.list 并建好工作区和目录。
- **谁处理哪类消息**：在配置里写 **bindings**，用 **match**（channel、accountId、peer 等）把入站消息路由到对应 **agentId**；最具体的规则优先，没有匹配则走默认 Agent。
- **渠道账号**：若每个 Agent 用不同机器人/不同号码，需在飞书/Telegram/WhatsApp 各建一套，并在 channels 里配好 accounts，再用 bindings 把 accountId 绑到 agentId。
- 改完配置要 **gateway restart**，用 **agents list --bindings** 和 **channels status --probe** 验证。

---

## 常见问题

**Q：bindings 写了但消息还是进了 main？**  
检查 match 是否写对：channel、accountId（若用多账号）、peer（若按人分）。**agents list --bindings** 看当前绑定是否生效；注意**最具体**的规则优先，若有多条匹配，按配置里顺序取第一条。

**Q：多个 Agent 能共用一个工作区吗？**  
不推荐。每个 Agent 应有**独立 workspace**，否则会话、人设、记忆会混在一起。若想「共享一部分设定」，可以复制 AGENTS.md/SOUL.md 到多个工作区再各自改，或把公共内容放到共享 Skill（`~/.openclaw/skills`）。

**Q：认证（API Key、OAuth）要每个 Agent 各配一份吗？**  
每个 Agent 的 **agentDir** 里各有 **auth-profiles.json**；默认不共享。若你希望某几个 Agent 用同一套模型认证，可以手动把一份 auth-profiles.json 复制到另一个 Agent 的 agentDir 里。不要多个 Agent 共用一个 agentDir，会乱。

**Q：飞书 / Telegram / WhatsApp 的 bindings 怎么写？**  
在 bindings 里写 `match: { channel: "feishu" }` 或 `"telegram"`、`"whatsapp"`；若该渠道配了多个 account，用 `accountId: "xxx"` 指定；若按发送者分，用 `peer: { kind: "direct", id: "..." }`。完整示例见官方 [Multi-Agent](https://docs.openclaw.ai/concepts/multi-agent) 的 Platform examples。

---

## 延伸阅读

- 多 Agent 路由与配置（英文）：[Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)  
- agents CLI（英文）：[agents](https://docs.openclaw.ai/cli/agents)  
- 工作区与多 Agent（英文）：[Agent workspace](https://docs.openclaw.ai/concepts/agent-workspace)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
