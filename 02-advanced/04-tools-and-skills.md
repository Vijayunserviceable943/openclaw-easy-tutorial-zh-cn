---
title: 教助手「新本领」：工具与 Skills
description: OpenClaw 内置工具、TOOLS.md、Skills 加载与编写、tools.allow/deny/profile、第三方 Skill 安全注意。
keywords: [OpenClaw, 工具, Tools, Skills, TOOLS.md, SKILL.md, 权限]
---

# 教助手「新本领」：工具与 Skills

读完这一节，你会知道助手能用的**内置工具有哪些**、**TOOLS.md** 到底管不管「能不能用」、**Skills** 从哪加载、怎样**自己写一个简单 Skill** 让助手学会新用法，以及如何用配置做**权限与安全**控制。适合想给助手加能力、又不想踩坑的你。

---

## 助手能用哪些「工具」

助手和外界打交道（读文件、执行命令、发消息、搜网页等）靠的是**工具**（Tools）：每个工具对应一种能力，模型在对话里会按需调用。OpenClaw 自带一整套内置工具，按功能可以粗分为几类（配置里常用 **group** 来批量放行或禁止）：

| 类别（group） | 包含的工具（举例） | 通俗理解 |
|---------------|--------------------|----------|
| **group:fs** | read, write, edit, apply_patch | 读、写、改工作区里的文件 |
| **group:runtime** | exec, bash, process | 在工作区里执行命令、管理后台进程 |
| **group:sessions** | sessions_list, sessions_history, sessions_send, sessions_spawn, session_status | 查会话、发消息到别的会话、派子 Agent 等 |
| **group:memory** | memory_search, memory_get | 查记忆文件（MEMORY.md、memory/ 日记） |
| **group:web** | web_search, web_fetch | 网页搜索、抓取页面内容 |
| **group:ui** | browser, canvas | 控制浏览器、画布等 |
| **group:messaging** | message | 往渠道（飞书、Telegram 等）发消息 |

还有 **group:automation**（cron、gateway）、**group:nodes**（节点设备）等；插件还会注册额外工具。完整列表和每个工具的参数见官方 [Tools](https://docs.openclaw.ai/tools)。

**谁决定「助手能不能用某个工具」？** 是**配置**里的 `tools.allow`、`tools.deny` 和 **tool profile**（见下），不是工作区里的 TOOLS.md。

---

## TOOLS.md 是干啥的

**TOOLS.md** 在工作区里，助手每次会话都会读到它；但它的作用是**给助手看的「使用说明」**，不是开关。

- 它**不决定**某个工具是否存在、能不能被调用；那是配置里 `tools.allow` / `tools.deny` / profile 的事。
- 它适合写：你本地有哪些脚本、约定怎么用 exec、怎么用 message 发到哪个渠道等。助手会参考这些说明去**正确使用**已经开放的工具，而不会因为 TOOLS.md 里没写就「不能用」某个工具。

所以：**开放/关闭工具 → 改配置**；**教助手怎么用、按什么约定用 → 改 TOOLS.md**。

---

## 用配置控制：谁能用哪些工具（权限与安全）

你可以在 `~/.openclaw/openclaw.json` 里做**全局**或**按 Agent** 的工具策略：

- **tools.allow** / **tools.deny**：白名单 / 黑名单，写工具名或 `group:xxx`（如 `group:fs`、`group:runtime`）。**deny 优先**：一旦 deny 了某个工具，助手就收不到该工具的调用入口。
- **tools.profile**：预设一组「基础允许列表」，再在上面用 allow/deny 微调。常见 profile：
  - **minimal**：只给很少工具（如 session_status），适合「只聊天、不干事」的场合。
  - **coding**：给文件、执行、会话、记忆、图片等，适合本地开发、跑命令。
  - **messaging**：偏向发消息、查会话，少文件/执行。
  - **full**：不额外限制（和没设 profile 类似）。

示例：只允许文件类 + 浏览器，禁止执行命令：

```json
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

示例：用 coding profile，但全局禁止执行类（exec、process 等）：

```json
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

多 Agent 时，可以在 `agents.list[].tools` 里给每个 Agent 单独设 profile 或 allow/deny；子 Agent、沙箱里也可以再套一层策略。详见官方 [Tools](https://docs.openclaw.ai/tools)、[Multi-Agent Sandbox & Tools](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools)。

**安全建议**：对不信任的输入或第三方 Skill，尽量用**沙箱**跑；不要把 exec 等高风险工具对所有人开放；API Key、密钥不要写进 Skill 说明里，用环境变量或配置里的 SecretRef。详见 [Security](https://docs.openclaw.ai/gateway/security)。

---

## Skills：教助手「怎么用」某类能力

**Skill**（技能）是一份「说明书」：告诉助手在什么场景下、用什么工具、按什么步骤做。它**不新增**工具实现，而是把**已有工具**的用法写清楚，让模型更会选、更会用。

- 一个 Skill = 一个**目录**，里面至少有一个 **SKILL.md** 文件。
- **SKILL.md** = 开头 YAML 元数据（name、description 等）+ 后面 Markdown 说明（何时用、用哪个工具、注意点等）。OpenClaw 兼容 [AgentSkills](https://agentskills.io) 的格式。

助手在会话里会看到「当前加载了哪些 Skill」的说明，从而更准确地在合适的时候调用对应工具。

---

## Skill 从哪加载、谁优先

Skills 从**三个来源**加载，**重名时**按下面优先级（先加载的会被后加载的同名覆盖，实际是「后面的优先」）：

1. **Bundled**：安装包自带的，优先级最低。
2. **Managed / 本机**：`~/.openclaw/skills`，所有 Agent 共用，优先级中。
3. **工作区**：`<工作区>/skills`（例如 `~/.openclaw/workspace/skills`），只对当前 Agent 可见，**优先级最高**。

你自建的 Skill 放在工作区的 `skills/` 下，就会覆盖同名的 bundled 或本机 Skill。还可以在配置里用 **skills.load.extraDirs** 加更多目录（优先级最低）。多 Agent 时，每个 Agent 的工作区各自有一套 `skills/`；`~/.openclaw/skills` 是共享的。

---

## 自己写一个简单 Skill

1. **建目录**（例如只在当前助手的工作区下用）：
   ```bash
   mkdir -p ~/.openclaw/workspace/skills/my-hello
   ```

2. **写 SKILL.md**：最少要有 **name** 和 **description**（YAML 头），后面用 Markdown 写「什么时候用、用哪个工具、怎么用」：
   ```markdown
   ---
   name: my_hello
   description: 用 echo 或 message 向用户问好
   ---

   # 我的问好技能

   当用户说「打个招呼」或「hello」时，用 `message` 工具（若已配置渠道）或简单回复一句「你好，我是你的助手！」。
   ```

3. **生效**：Gateway 会在加载时扫描 skills 目录；新加或改完 Skill 后，**新开一个会话**或重启 Gateway，助手就能读到新内容。不需要改配置，只要目录和 SKILL.md 在约定位置即可。

更多元数据（如 **metadata.openclaw.requires.bins**、**requires.env** 用于「本机有某命令/某环境变量才加载」）、Slash 命令等见官方 [Skills](https://docs.openclaw.ai/tools/skills)、[Creating Skills](https://docs.openclaw.ai/tools/creating-skills)。

---

## 用 ClawHub 安装别人写好的 Skill

[ClawHub](https://clawhub.com) 是 OpenClaw 的公开 Skill 仓库。你可以用命令行安装、更新：

- 安装到当前工作区的 `skills/` 下：`clawhub install <skill-slug>`
- 更新已安装的：`clawhub update --all`

默认会装到当前目录下的 `./skills`（或配置里的工作区）；装好后 OpenClaw 会当作工作区 skills 加载。**第三方 Skill 要当不可信代码**：装前看一眼 SKILL.md 和里面有没有调高风险工具，必要时用沙箱或限制工具 profile。

---

## 本节要点

- **内置工具**分很多类（文件、执行、会话、记忆、网页、浏览器、消息等），用 **group:xxx** 或具体工具名在配置里 allow/deny。
- **TOOLS.md** 只负责「说明怎么用」，不负责「开/关」工具；开关在 **tools.allow**、**tools.deny**、**tools.profile**。
- **工具权限**：用 profile（minimal/coding/messaging/full）+ allow/deny；多 Agent 可用 **agents.list[].tools**；建议高风险场景用沙箱、限制 exec。
- **Skill** = 目录 + **SKILL.md**（YAML 头 + Markdown 说明），教助手何时用哪个工具；加载顺序：bundled &lt; 本机 `~/.openclaw/skills` &lt; 工作区 `skills/`。
- **自建 Skill**：在工作区下建 `skills/技能名/`，写 SKILL.md，新会话或重启后生效；可用 **ClawHub** 安装他人 Skill，注意安全、必要时沙箱。

---

## 常见问题

**Q：TOOLS.md 里写了某个工具，为什么助手还是不用？**  
TOOLS.md 只是提示，模型可能选别的做法；且若该工具在配置里被 **deny** 或在 profile 里没被放行，助手根本拿不到该工具，自然不会用。先确认配置里该工具已允许。

**Q：Skill 改完了没反应？**  
确认 SKILL.md 在正确目录（工作区 `skills/技能名/SKILL.md` 或 `~/.openclaw/skills/技能名/SKILL.md`），且**新开了一个会话**（或重启了 Gateway）；同一会话内可能已经加载过旧版本。

**Q：想禁止助手执行任何命令？**  
在配置里设 `tools.deny: ["group:runtime"]`（或 `["exec", "bash", "process"]`）；若用 profile，可选 `tools.profile: "messaging"` 再按需 allow 个别工具。

**Q：第三方 Skill 安全吗？**  
要当**不可信**处理：先看 SKILL.md 里有没有调 exec、写文件、发消息等；再考虑用沙箱、或给该 Agent 用更小的 tools.profile。不要把密钥写进 Skill 正文；用环境变量或配置 SecretRef。详见 [Security](https://docs.openclaw.ai/gateway/security)。

---

## 延伸阅读

- 工具总览与配置（英文）：[Tools](https://docs.openclaw.ai/tools)  
- Skills 加载与格式（英文）：[Skills](https://docs.openclaw.ai/tools/skills)  
- 自建 Skill 步骤（英文）：[Creating Skills](https://docs.openclaw.ai/tools/creating-skills)  
- ClawHub 使用（英文）：[ClawHub](https://docs.openclaw.ai/tools/clawhub)  
- 多 Agent 下的工具与沙箱（英文）：[Multi-Agent Sandbox & Tools](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools)  
- 安全与威胁模型（英文）：[Security](https://docs.openclaw.ai/gateway/security)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
