---
title: 案例：律师 + 会计师 — 多角色聊「诉讼与合规」
description: 用 OpenClaw 多 Agent 与子 Agent 搭建律师、会计师角色，同一讨论中获取诉讼合规与财税意见并综合。legal、accountant 人设与 spawn。
keywords: [OpenClaw, 多角色, 律师, 会计师, 诉讼, 合规, 子 Agent, legal, accountant]
---

# 案例：律师 + 会计师 — 多角色聊「诉讼与合规」

这一节是一个**动手案例**：用 OpenClaw 的**多 Agent** 和**子 Agent**，搭出「律师」和「会计师」两个角色，在同一次讨论里分别从**诉讼与合规**、**财税与账务**角度给意见，最后你或主助手综合两边的结论。用到的都是前面 3.1、3.2 已讲过的能力，不编造新功能。

---

## 你想达成的效果

假设你是甲方，有一笔交易或一份合同，想同时听：

- **律师**：从诉讼风险、合规、条款解释给意见；
- **会计师**：从税务处理、账务合规、报表影响给意见。

在 OpenClaw 里可以这样实现：建两个**独立 Agent**（legal、accountant），各自有**独立工作区**和人设（AGENTS.md、SOUL.md）；你在**主会话**里描述案情或贴条款，然后**派两个子 Agent** 分别用「律师」和「会计师」的身份回答，等两份结果都回来后再一起看、或让主助手帮你总结。这就是本案例的「多角色讨论」形态。

---

## 前置：你已经会的内容

- **3.1**：会用 `openclaw agents add` 加新 Agent，知道每个 Agent 有独立工作区、agentDir、会话。
- **3.2**：会用 `/subagents spawn 某个agentId 任务` 或主助手用 **sessions_spawn** 派子 Agent，知道子 Agent 跑完会 **announce** 回当前对话。
- 会改工作区里的 **AGENTS.md**、**SOUL.md** 定人设（见 1.4）。

若还没做过 3.1，先按 3.1 建好 **legal**、**accountant** 两个 Agent，再往下做。

---

## 第一步：建两个 Agent 并定好人设

若还没有「律师」「会计师」两个 Agent，在终端执行：

```bash
openclaw agents add legal
openclaw agents add accountant
```

每个 Agent 会有一个独立工作区，例如 `~/.openclaw/workspace-legal`、`~/.openclaw/workspace-accountant`。接下来给**各自**的工作区写好人设，让它们真的像「律师」和「会计师」在说话。

**律师（legal）**  
编辑 `~/.openclaw/workspace-legal/SOUL.md`（或你配置里 legal 的 workspace 路径），例如：

```markdown
- 身份：扮演一名谨慎、专业的法律顾问，侧重诉讼风险与合规。
- 语气：简洁、条理清晰，必要时分点列出风险与建议。
- 边界：只做一般性分析与提示，不代替执业律师出具正式法律意见；涉及具体司法辖区时提醒用户咨询当地律师。
```

在 **AGENTS.md** 里可以加几条工作规则，例如：回答时先概括结论，再分点说明理由；涉及金额、期限、主体时务必指出来。

**会计师（accountant）**  
编辑 `~/.openclaw/workspace-accountant/SOUL.md`，例如：

```markdown
- 身份：扮演一名财税与账务合规顾问，侧重税务处理、记账与报表影响。
- 语气：用词准确，涉及科目、税率、时点处说清楚。
- 边界：只做一般性分析，不代替执业会计师出具鉴证意见；涉及具体税种、地区政策时提醒用户咨询当地会计师。
```

同样在 **AGENTS.md** 里约定：回答时区分「税务」「账务」「报表」等维度；涉及数字时注明假设与适用口径。

这样，**legal** 和 **accountant** 两个 Agent 就各自有了「律师」「会计师」的人设；后面派子 Agent 时，用这两个 agentId，就会用这两套人设回复。

---

## 第二步：允许主会话派子 Agent 给 legal、accountant

你要在**主会话**里（例如和 main 聊天时）能派子 Agent 给 **legal** 和 **accountant**，需要在配置里放开 **subagents.allowAgents**。

编辑 `~/.openclaw/openclaw.json`，在 **main** 这个 Agent 的配置里加上（若你是用 main 当「主持人」、和它对话的那一个）：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "subagents": { "allowAgents": ["legal", "accountant"] }
      },
      { "id": "legal", "workspace": "~/.openclaw/workspace-legal" },
      { "id": "accountant", "workspace": "~/.openclaw/workspace-accountant" }
    ]
  }
}
```

这样，当你在「main」的会话里派子 Agent 时，就可以指定 **agentId** 为 legal 或 accountant。若你的 agents.list 结构已经存在，只要在 main 对应的那一项里加上 **subagents.allowAgents** 即可。保存后执行 `openclaw gateway restart`。

---

## 第三步：在同一次讨论里拿到「律师」和「会计师」的意见

有两种用法，二选一或混用即可。

**方式 A：你直接派两次子 Agent（适合 Control UI 或支持斜杠命令的渠道）**

在你和 main（或当前默认 Agent）的对话里，先发一段案情或合同摘要，然后发两条斜杠命令，分别让「律师」和「会计师」从各自角度给意见，例如：

```
/subagents spawn legal 从诉讼与合规角度，对以下交易/条款的风险与建议给出简要意见：（这里粘贴或简述案情/条款）
/subagents spawn accountant 从财税与账务合规角度，对同一交易/条款的税务处理、记账与报表影响给出简要意见：（同一段案情/条款）
```

两条都会**非阻塞**执行；等两条都跑完，你会先后收到两条 **announce** 汇报（律师结论、会计师结论）。你可以在同一对话里再问 main：「请根据上面律师和会计师的意见，帮我总结一份要点」，由 main 综合两边的结论。

**方式 B：让主助手帮你派（口述即可）**

你也可以不对着斜杠命令，直接对主助手说，例如：

「我们模拟一场诉讼与合规讨论：我是甲方，有一段交易/合同（简述）。请你先派律师 Agent 从诉讼与合规角度给意见，再派会计师 Agent 从财税角度给意见；等两边都回复后，你再帮我综合成一份简要结论。」

主助手若具备 **sessions_spawn** 能力且 **allowAgents** 里包含 legal、accountant，就会在对话里调用两次 sessions_spawn（一次 agentId=legal，一次 agentId=accountant），等两次子 Agent 的 announce 都回来后再综合回复。这样你不需要自己打斜杠命令，也能完成「律师 + 会计师」多角色讨论。

---

## 第四步：根据两份结果做小结

- 若你用**方式 A**：两条 announce 会出现在当前对话里，你可以自己对比阅读，或再发一条消息让 main「根据上面律师和会计师的回复，总结三点结论」。
- 若你用**方式 B**：主助手会在收到两份 announce 后，在同一轮回复里做综合；你只需看主助手的那条总结即可。

至此，一次「律师 + 会计师」多角色诉讼与合规讨论就完成了。重复使用时，只需换案情或条款内容，再同样派 legal、accountant 即可。

---

## 本节要点

- **目标**：在同一讨论里同时拿到「律师」（诉讼与合规）和「会计师」（财税与账务）两个角色的意见，用 OpenClaw 多 Agent + 子 Agent 实现。
- **准备**：用 `openclaw agents add legal`、`openclaw agents add accountant` 建两个 Agent；在**各自工作区**的 SOUL.md、AGENTS.md 里写好律师/会计师人设与边界。
- **权限**：在主会话对应的 Agent（如 main）上配置 **subagents.allowAgents**，包含 `["legal", "accountant"]`，重启 Gateway。
- **用法**：要么你发 **/subagents spawn legal ...** 和 **/subagents spawn accountant ...** 分别派活，等两条 announce 后自己或让 main 综合；要么口述让主助手「先派律师、再派会计师，再综合」，由主助手调 sessions_spawn 完成。不编造能力，全部基于 3.1、3.2 的已有功能。

---

## 常见问题

**Q：主助手没有自动派 legal/accountant，而是自己答了？**  
确认 main（或你对话所在 Agent）的配置里 **subagents.allowAgents** 已包含 legal、accountant，且已重启 Gateway；再确认你对主助手下的指令里明确说了「派律师 Agent」「派会计师 Agent」或「用 sessions_spawn 让 legal/accountant 回答」。若主助手没有 sessions_spawn 权限或没被提示用子 Agent，它可能直接自己答。

**Q：legal / accountant 的回复太泛，不像律师/会计师？**  
把人设写细一点：在各自工作区的 **SOUL.md** 里强调身份、语气、边界；在 **AGENTS.md** 里写「先给结论再分点」「区分诉讼风险与财税影响」等规则。任务描述也可以写清楚：「从**诉讼与合规**角度」「从**财税与账务**角度」，并贴具体条款或案情，便于模型紧扣角色回答。

**Q：可以同时派多个子 Agent 吗？**  
可以。你连续发两条 `/subagents spawn legal ...` 和 `/subagents spawn accountant ...` 会得到两个子 Agent 并行跑；或主助手在一次回复里调两次 sessions_spawn，也会并行。两边都跑完后会有两条 announce，再综合即可。

---

## 延伸阅读

- 多 Agent 搭建（本教程）：[3.1 搭建多 Agent 环境](01-multi-agent-setup.md)  
- 子 Agent 与并行任务（本教程）：[3.2 子 Agent 与并行任务](02-subagents-and-tasks.md)  
- 官方 Multi-Agent（英文）：[Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)  
- 官方 Sub-Agents（英文）：[Sub-Agents](https://docs.openclaw.ai/tools/subagents)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
