---
title: 案例：产品 + 开发 + 测试 — 多角色完成一个小项目
description: 用 OpenClaw 多 Agent 与子 Agent 搭建产品、开发、测试角色，从需求到实现到测试要点协作完成小项目推演。product、coding、qa 配置。
keywords: [OpenClaw, 多角色, 产品, 开发, 测试, 用户故事, 子 Agent, product, coding, qa]
---

# 案例：产品 + 开发 + 测试 — 多角色完成一个小项目

这一节是一个**动手案例**：用 OpenClaw 的**多 Agent** 和**子 Agent**，搭出「产品」「开发」「测试」三个角色，围绕一个小型软件需求，从**需求整理**到**实现思路**再到**测试要点**分工协作，最后你或主助手综合三边的输出。用到的同样是 3.1、3.2 的能力，不编造新功能。

---

## 你想达成的效果

假设你有一个小需求或一个小功能点子，希望快速得到：

- **产品**：把需求整理成用户故事、验收标准，避免歧义；
- **开发**：从实现角度给出技术方案或代码草图；
- **测试**：从质量角度列出测试点、边界情况、回归注意点。

在 OpenClaw 里可以这样实现：建三个**独立 Agent**（product、coding、qa），各自有**独立工作区**和人设；你在**主会话**里描述需求，然后**派三个子 Agent** 分别用「产品」「开发」「测试」的身份产出，等三份结果都回来后再一起看、或让主助手帮你汇总成一份「小项目清单」。这就是本案例的「多角色完成一个小项目」形态。

---

## 前置：你已经会的内容

- **3.1**：会用 `openclaw agents add` 加新 Agent，知道每个 Agent 有独立工作区、agentDir、会话。
- **3.2**：会用 `/subagents spawn 某个agentId 任务` 或主助手用 **sessions_spawn** 派子 Agent，知道子 Agent 跑完会 **announce** 回当前对话。
- 会改工作区里的 **AGENTS.md**、**SOUL.md** 定人设（见 1.4）。

若还没做过 3.1，先按 3.1 建好 **product**、**coding**、**qa** 三个 Agent，再往下做。

---

## 第一步：建三个 Agent 并定好人设

在终端执行：

```bash
openclaw agents add product
openclaw agents add coding
openclaw agents add qa
```

每个 Agent 会有一个独立工作区，例如 `~/.openclaw/workspace-product`、`~/.openclaw/workspace-coding`、`~/.openclaw/workspace-qa`。接下来给**各自**的工作区写好人设，让它们真的像「产品」「开发」「测试」在说话。

**产品（product）**  
编辑 `~/.openclaw/workspace-product/SOUL.md`，例如：

```markdown
- 身份：扮演产品角色，侧重需求澄清、用户故事与验收标准。
- 语气：条理清晰，用「作为…我希望…以便…」等形式写用户故事；验收标准可测、无歧义。
- 边界：只做需求与范围梳理，不代替实际项目管理；涉及优先级与排期时提醒用户自行决策。
```

在 **AGENTS.md** 里可以约定：先给 1～3 条用户故事，再列每条对应的验收标准；涉及接口或数据时写清输入输出预期。

**开发（coding）**  
编辑 `~/.openclaw/workspace-coding/SOUL.md`，例如：

```markdown
- 身份：扮演开发角色，侧重技术方案、实现思路与代码级建议。
- 语气：简洁技术向，必要时给出伪代码或示例代码；标明技术栈与依赖假设。
- 边界：只做方案与示例，不代替完整工程实现；涉及部署、运维时提醒用户结合自身环境。
```

在 **AGENTS.md** 里可以约定：先简述实现思路，再给关键步骤或代码片段；如需读写文件、调用 API，写清前提条件。

**测试（qa）**  
编辑 `~/.openclaw/workspace-qa/SOUL.md`，例如：

```markdown
- 身份：扮演测试角色，侧重测试点、边界与回归影响。
- 语气：用检查清单或条目列出；区分功能、边界、异常、回归等维度。
- 边界：只做测试思路与要点，不代替实际测试执行；涉及自动化时提醒用户按项目选型。
```

在 **AGENTS.md** 里可以约定：先列功能/正向测试点，再列边界与异常，最后提一句回归或影响范围。

这样，**product**、**coding**、**qa** 三个 Agent 就各自有了清晰角色；后面派子 Agent 时用这三个 agentId，就会用这三套人设回复。

---

## 第二步：允许主会话派子 Agent 给 product、coding、qa

你要在**主会话**里（例如和 main 聊天时）能派子 Agent 给这三个角色，需要在配置里放开 **subagents.allowAgents**。

编辑 `~/.openclaw/openclaw.json`，在 **main** 对应的那一项里加上（若你是用 main 当「项目协调人」、和它对话的那一个）：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "subagents": { "allowAgents": ["product", "coding", "qa"] }
      },
      { "id": "product", "workspace": "~/.openclaw/workspace-product" },
      { "id": "coding", "workspace": "~/.openclaw/workspace-coding" },
      { "id": "qa", "workspace": "~/.openclaw/workspace-qa" }
    ]
  }
}
```

若你的 agents.list 里已有 main、product、coding、qa，只需在 main 上加上 **subagents.allowAgents: ["product", "coding", "qa"]**。保存后执行 `openclaw gateway restart`。

---

## 第三步：用「产品 + 开发 + 测试」跑完一个小需求

有两种用法，可二选一或混用。

**方式 A：你直接派三个子 Agent（适合 Control UI 或支持斜杠命令的渠道）**

在你和 main 的对话里，先发一段需求描述，然后发三条斜杠命令，分别让「产品」「开发」「测试」从各自角度产出，例如：

```
/subagents spawn product 把下面需求整理成 1～3 条用户故事和验收标准：（粘贴或简述需求）
/subagents spawn coding 针对同一需求，给出实现思路和关键代码或伪代码：（同一段需求）
/subagents spawn qa 针对同一需求，列出测试点、边界与异常、回归注意点：（同一段需求）
```

三条都会**非阻塞**执行；等三条都跑完，你会先后收到三条 **announce** 汇报（产品输出、开发输出、测试输出）。你可以在同一对话里再问 main：「请根据上面产品、开发、测试的回复，帮我整理成一份小项目清单（需求 + 实现要点 + 测试要点）」，由 main 综合三边的结论。

**方式 B：让主助手帮你派（口述即可）**

你也可以直接对主助手说，例如：

「我们做一个小项目推演：我有一个需求（简述）。请你先派产品 Agent 整理成用户故事和验收标准，再派开发 Agent 给实现思路和关键代码，再派测试 Agent 给测试点和回归注意点；等三边都回复后，你再帮我综合成一份简要的项目清单。」

主助手若具备 **sessions_spawn** 能力且 **allowAgents** 里包含 product、coding、qa，就会在对话里调用三次 sessions_spawn，等三次子 Agent 的 announce 都回来后再综合回复。这样你不需要自己打三条斜杠命令，也能完成「产品 + 开发 + 测试」多角色小项目推演。

---

## 第四步：根据三份结果做小结

- 若你用**方式 A**：三条 announce 会出现在当前对话里，你可以自己对比阅读，或再发一条消息让 main「根据上面产品、开发、测试的回复，总结成一份项目清单」。
- 若你用**方式 B**：主助手会在收到三份 announce 后，在同一轮回复里做综合；你只需看主助手的那条总结即可。

至此，一次「产品 + 开发 + 测试」多角色完成一个小项目的推演就完成了。重复使用时，只需换需求描述，再同样派 product、coding、qa 即可；你也可以只派其中一两个角色（例如只派产品和开发），按需组合。

---

## 本节要点

- **目标**：围绕一个小型软件需求，同时拿到「产品」（用户故事与验收标准）、「开发」（实现思路与代码要点）、「测试」（测试点与回归注意）三个角色的输出，用 OpenClaw 多 Agent + 子 Agent 实现。
- **准备**：用 `openclaw agents add product`、`openclaw agents add coding`、`openclaw agents add qa` 建三个 Agent；在**各自工作区**的 SOUL.md、AGENTS.md 里写好产品/开发/测试人设与产出格式。
- **权限**：在主会话对应的 Agent（如 main）上配置 **subagents.allowAgents**，包含 `["product", "coding", "qa"]`，重启 Gateway。
- **用法**：要么你发 **/subagents spawn product ...**、**/subagents spawn coding ...**、**/subagents spawn qa ...** 分别派活，等三条 announce 后自己或让 main 综合；要么口述让主助手「先派产品、再派开发、再派测试，再综合」，由主助手调 sessions_spawn 完成。不编造能力，全部基于 3.1、3.2 的已有功能。

---

## 常见问题

**Q：主助手没有自动派 product/coding/qa，而是自己答了？**  
确认 main（或你对话所在 Agent）的配置里 **subagents.allowAgents** 已包含 product、coding、qa，且已重启 Gateway；再确认你对主助手下的指令里明确说了「派产品 Agent」「派开发 Agent」「派测试 Agent」或「用 sessions_spawn 让 product/coding/qa 分别回答」。若主助手没有 sessions_spawn 权限或没被提示用子 Agent，它可能直接自己答。

**Q：三个角色的回复风格不够区分？**  
把人设写细一点：在各自工作区的 **SOUL.md** 里强调身份、产出形式（用户故事 vs 代码 vs 测试清单）；在 **AGENTS.md** 里写「先用户故事再验收标准」「先实现思路再代码」「先功能点再边界再回归」等规则。任务描述也可以写清楚：「把下面需求整理成用户故事和验收标准」「针对同一需求给实现思路和关键代码」「针对同一需求列测试点和回归注意点」，并贴同一段需求，便于模型紧扣角色回答。

**Q：可以只派其中两个角色吗？**  
可以。例如只派 product 和 coding，不做测试推演；或先派 product 产出需求，再根据需求派 coding 和 qa。按你的实际需要组合即可。

**Q：coding 能直接在我项目里写代码吗？**  
本案例里，coding 只是在**对话里**给实现思路和代码示例；若你要它在真实项目目录里读写文件，需要让该 Agent 具备文件类工具权限，并在其工作区或任务描述里指明路径（参见 2.4 工具与 Skills）。多角色协作本身不要求 coding 必须动你本机文件，先跑通「需求 → 实现要点 → 测试要点」的推演流程即可。

---

## 延伸阅读

- 多 Agent 搭建（本教程）：[3.1 搭建多 Agent 环境](01-multi-agent-setup.md)  
- 子 Agent 与并行任务（本教程）：[3.2 子 Agent 与并行任务](02-subagents-and-tasks.md)  
- 律师 + 会计师案例（本教程）：[3.3 案例：律师 + 会计师](03-case-multi-role-legal.md)  
- 官方 Multi-Agent（英文）：[Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)  
- 官方 Sub-Agents（英文）：[Sub-Agents](https://docs.openclaw.ai/tools/subagents)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
