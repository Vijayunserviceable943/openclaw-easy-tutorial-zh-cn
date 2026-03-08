---
title: 日常怎么用、出问题怎么查
description: OpenClaw 日常使用、Gateway 状态、日志查看、重启与更新；连不上、收不到消息、报错时的排查思路与文档入口。
keywords: [OpenClaw, 日常使用, 日志, gateway status, 故障排查, 更新]
---

# 日常怎么用、出问题怎么查

读完这一节，你会知道**平时**怎么打开助手、看运行状态和日志，需要时怎么**重启**或**更新**；遇到「连不上、收不到消息、报错」时，该先查哪几样、去哪里找更详细的说明。都是最常用、最省事的做法。

---

## 日常怎么用

### 和助手聊天

- **在浏览器里用**：在终端执行 `openclaw dashboard`，或直接在浏览器打开 `http://127.0.0.1:18789/`（或 `http://localhost:18789/`）。这就是 **Control UI**（控制台），不接飞书/Telegram/WhatsApp 也能随时和助手对话。
- **在飞书 / Telegram / WhatsApp 里用**：只要 Gateway 在运行、渠道已按前面章节接好，直接在这些 App 里给助手发消息即可。

### 看 Gateway 有没有在跑

在终端执行：

```bash
openclaw gateway status
```

若显示在运行（例如 `Runtime: running`、`RPC probe: ok`），说明那座「桥」正常；若没有，需要先启动或重启（见下文「重启 Gateway」）。

### 看日志（出问题时很有用）

助手和 Gateway 的运行记录会写到日志里。在终端执行：

```bash
openclaw logs --follow
```

会**持续**把最新日志打在你眼前（按 Ctrl+C 停止）。不加 `--follow` 就只拉最近一段。若你用的是**远程**机器上的 OpenClaw，这条命令也会通过 RPC 把远程日志拉过来，不用登进那台机器看文件。

---

## 需要时：重启 Gateway

改过配置、装过插件，或觉得「卡住了、不对劲」时，可以先试着重启 Gateway。

若你当初用 `openclaw onboard --install-daemon` 装过**后台服务**，在终端执行：

```bash
openclaw gateway restart
```

（老文档里可能写成 `openclaw daemon restart`，效果一样：都是重启 Gateway 进程。）

若你是**手动**在终端里用 `openclaw gateway --port 18789` 跑的，没有装成服务，那就先在那台终端按 Ctrl+C 停掉，再重新执行一次 `openclaw gateway --port 18789` 即可。

---

## 需要时：更新 OpenClaw

OpenClaw 还在快速迭代，偶尔更新能用到新功能和修复。推荐方式：**再跑一遍安装脚本**，会在原有基础上升级，一般不会清空你的配置和工作区。

在终端执行（macOS/Linux）：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

若不想再走一遍向导，可加 `--no-onboard`：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
```

若你是用 **npm** 全局装的（`npm install -g openclaw`），也可以直接：

```bash
npm i -g openclaw@latest
```

更新完成后，若 Gateway 是以后台服务方式跑的，一般需要**重启一次**（`openclaw gateway restart`）。用安装脚本时，脚本有时会帮你跑检查或重启；用 npm 更新时记得自己重启。

更新前建议心里有数：你的配置在 `~/.openclaw/openclaw.json`，工作区在 `~/.openclaw/workspace`，这些通常不会被覆盖，但若有重要改动，先备份一份更安心。

---

## 出问题怎么查：按顺序来

遇到「连不上、收不到消息、报错」时，可以**按下面顺序**做几项检查，多数情况能定位到方向。

### 1. 先看整体状态

在终端依次执行：

```bash
openclaw gateway status
openclaw doctor
```

- **gateway status**：看 Gateway 是否在跑、RPC 是否通。
- **doctor**：会检查配置、凭证、工作区等是否正常，并给出可执行的修复建议（例如配置过期、多余的工作区目录等）。直接执行 `openclaw doctor` 即可，按提示选择是否自动修复。

### 2. 控制台 / 网页打不开（连不上）

- 确认 Gateway 在跑：`openclaw gateway status`。若没在跑，用上面「重启 Gateway」的方式启动。
- 确认地址没错：本机用 `http://127.0.0.1:18789/` 或 `http://localhost:18789/`（端口以你配置为准，默认 18789）。
- 若从**别的电脑**访问：要保证那台电脑能访问到运行 OpenClaw 的机器（网络、防火墙、认证）。第一次从新设备连时，可能需要在运行 OpenClaw 的机器上执行 `openclaw devices list` 和 `openclaw devices approve <requestId>` 做设备配对。

### 3. 飞书 / Telegram / WhatsApp 收不到回复

- **先看渠道状态**：执行 `openclaw channels status --probe`，看对应渠道是否已连接、就绪。
- **配对 / 白名单**：若你用的是**配对**（pairing），陌生人第一次发消息只会收到配对码，需要你在本机执行：
  - `openclaw pairing list <渠道名>`（如 `telegram`、`whatsapp`、`feishu`）
  - `openclaw pairing approve <渠道名> <配对码>`
  批准后才会正常回复。若你设的是**白名单**（allowFrom），确认发消息的号码或账号在名单里。
- **看日志**：`openclaw logs --follow`，看有没有报错（例如 API 失败、认证失败、网络错误）。根据错误信息再查对应渠道的官方文档（见下方「延伸阅读」）。

### 4. 权限或配置报错

- 先跑一遍 **doctor**：`openclaw doctor`，看是否有「配置过期、需要迁移」等提示，按提示选修复。
- 若提示和**允许名单、sender ID** 有关：可能是升级后配置格式有变化，可查 [Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting) 或对应渠道页的 Troubleshooting；有时需要把 `@用户名` 改成数字 ID，或重新批准配对。

---

## 哪里查更详细的文档

- **官方文档（英文）**：[docs.openclaw.ai](https://docs.openclaw.ai) — 安装、配置、各渠道、CLI 命令、故障排查都有。
- **多语言文档站（对非英文用户更友好）**：[www.openclawx.cloud](https://www.openclawx.cloud) — 多语种版本，结构和官方一致。
- **本教程附录**：常见问题集中整理在 [附录 A：常见问题速查](../appendix/faq.md)；官方与多语言站链接在 [附录 B：官方文档与资源](../appendix/references.md)。

遇到具体报错时，可以把错误信息或日志里关键几行复制下来，在官方站或本仓库的 Issues 里搜一搜，往往能找到同类问题和解法。

---

## 本节要点

- **日常**：用 `openclaw dashboard` 或浏览器打开 `http://127.0.0.1:18789/` 和助手聊天；用 `openclaw gateway status` 看是否在跑；用 `openclaw logs --follow` 看实时日志。
- **重启**：装过后台服务用 `openclaw gateway restart`；手动跑的就在终端 Ctrl+C 后重新执行 `openclaw gateway`。
- **更新**：建议再跑一遍安装脚本（可加 `--no-onboard`），或用 `npm i -g openclaw@latest`；更新后记得重启 Gateway。
- **排错顺序**：先 `openclaw gateway status` 和 `openclaw doctor`，再根据「连不上 / 收不到消息 / 权限报错」查控制台与端口、配对与白名单、渠道 probe 与日志；详细步骤查官方 [Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting) 或 [www.openclawx.cloud](https://www.openclawx.cloud)。

---

## 常见问题

**Q：`openclaw gateway status` 说没在运行，怎么办？**  
若你装过后台服务：执行 `openclaw gateway start` 再查一次。若从没装过服务，就在终端执行 `openclaw gateway --port 18789` 保持运行；关掉终端会停掉，需要时再开。

**Q：更新后控制台或渠道异常？**  
先执行 `openclaw gateway restart`。若仍异常，跑一遍 `openclaw doctor`，按提示做修复或迁移；再查 [Updating](https://docs.openclaw.ai/install/updating) 和对应渠道的 Troubleshooting。

**Q：日志太多、不知道从哪看起？**  
用 `openclaw logs --follow` 实时看，再在另一个终端复现问题（例如发一条消息），看同一时刻日志里出现的报错或警告；把那段贴到搜索引擎或官方文档里搜，往往能定位到原因。

**Q：想确认渠道到底连没连上？**  
执行 `openclaw channels status --probe`，会逐个检查已配置的渠道连接状态；若有报错，再结合 `openclaw logs --follow` 和该渠道官方文档的 Troubleshooting 排查。

---

## 延伸阅读

- Gateway 与状态查询（英文）：[gateway](https://docs.openclaw.ai/cli/gateway)  
- 日志命令（英文）：[logs](https://docs.openclaw.ai/cli/logs)  
- 更新与维护（英文）：[Updating](https://docs.openclaw.ai/install/updating)  
- 配置与状态修复（英文）：[Doctor](https://docs.openclaw.ai/gateway/doctor)  
- 渠道故障排查（英文）：[Channel troubleshooting](https://docs.openclaw.ai/channels/troubleshooting)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
