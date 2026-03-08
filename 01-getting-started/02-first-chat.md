---
title: 十分钟内：安装并在浏览器里和助手说上话
description: 十分钟完成 OpenClaw 安装与向导配置，在浏览器控制台与助手完成第一次对话。Node、一键安装、Control UI 入门。
keywords: [OpenClaw, 安装, 十分钟, 控制台, Control UI, Node, 向导]
---

# 十分钟内：安装并在浏览器里和助手说上话

读完这一节并照着做，你会在**十分钟内**完成 OpenClaw 的安装、跟着向导做完基本配置，并在**浏览器里**和助手聊上第一句。不用接飞书、Telegram 或 WhatsApp，也能先玩起来——所有对话都在网页里的「控制台」完成。

---

## 你需要先有的

- 一台电脑（macOS、Linux 或 Windows；Windows 建议用 **WSL2**，见下方常见问题）。
- **Node 22 或更高**：若不确定，在终端输入 `node --version` 查看；没有或版本太低时，下面的一键安装脚本在 macOS/Linux 上会**顺带帮你装好 Node**。

---

## 第一步：一键安装 OpenClaw

打开**终端**（macOS/Linux）或 **PowerShell**（Windows），复制下面对应你系统的**一整行**命令，粘贴到终端里，回车执行。

**macOS / Linux / WSL2：**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows（PowerShell）：**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

脚本会下载并安装 OpenClaw 的**命令行工具**（简称 **CLI**：你在终端里输入 `openclaw` 开头的命令，就是通过它和 OpenClaw 打交道）。在 macOS/Linux 上，若检测到你还没装 Node 或版本不够，脚本会先帮你装好 Node 22+，再装 OpenClaw。

> **提示**：若你只想先装好软件、稍后再自己跑配置，可以在命令后加 `--no-onboard`（仅安装，不启动向导）。本教程建议第一次直接不加，让脚本带你走完下一步的向导。

安装完成后，在终端输入 `openclaw` 并回车，若能看到用法说明而不是「找不到命令」，就说明装好了。

---

## 第二步：运行「向导」，完成基本配置

安装好后，需要做一次**一次性配置**：选一个 AI 模型、设置 Gateway（那座「桥」）的端口和认证、以及助手用的「工作区」文件夹等。官方提供了**向导**（onboarding wizard），会一步步问你，你只要按提示选或填即可。

在终端执行：

```bash
openclaw onboard --install-daemon
```

- **onboard**：启动配置向导，会引导你完成模型/认证、Gateway、工作区、可选渠道（如 Telegram、WhatsApp）等。
- **--install-daemon**：配置完成后，把 OpenClaw 的 **Gateway** 装成**后台服务**（daemon），这样你关掉终端后它也会继续运行，随时能接收消息或让你在浏览器里聊天。

向导里若问你要不要接某个**渠道**（Channel，即飞书、Telegram、WhatsApp 等），可以**先全部跳过**。我们这一节的目标是：先在浏览器里和助手说上话，不接任何 IM 也能用。

> **小知识**：**工作区**（Workspace）是助手「上班的文件夹」，里面会有人设、规则、记忆等文件；默认在 `~/.openclaw/workspace`。向导会帮你创建好并放入初始文件，后面章节会教你怎么改。

---

## 第三步：确认 Gateway 已在运行

配置完成后，Gateway 一般已经以后台服务形式跑起来了。在终端执行：

```bash
openclaw gateway status
```

若显示正在运行，就说明那座「桥」已经就绪。若没有，可以手动启动一次（见下方「常见问题」）。

---

## 第四步：在浏览器里打开「控制台」，和助手聊第一句

OpenClaw 提供了一个**网页版控制台**，官方叫 **Control UI**（控制界面）。你可以把它理解成：Gateway 自带的「小网站」，用来和助手对话、看会话、做简单设置。它和 Gateway 在同一个端口上，默认是 **18789**。

在终端执行：

```bash
openclaw dashboard
```

一般会自动打开浏览器并跳到控制台页面。若没有自动打开，请自己在浏览器地址栏输入：

```
http://127.0.0.1:18789/
```

（或 `http://localhost:18789/`，效果一样。）

第一次打开时，若提示需要**配对**或输入**令牌**（token）：向导在配置时已经生成了一个 Gateway 认证用的 token，在向导结束时的输出里会有一行类似 `gateway.auth.token` 的值，把它复制到控制台里要求填 token 的地方即可。若你是在**本机**用浏览器访问 `127.0.0.1`，很多情况下会**自动通过**，不会卡住。

进入页面后，找到和助手对话的输入框，随便发一句「你好」或问一个问题，就能完成**第一次对话**。

---

## 完成：你已经和助手说上话了

若控制台能打开、能发消息并收到助手回复，就说明：

- OpenClaw 已正确安装；
- Gateway 在运行；
- 你已经可以在**不接任何飞书/Telegram/WhatsApp** 的情况下，用浏览器和助手日常聊天。

下一步你可以：继续用控制台玩一段时间，或者跟着下一节「让助手出现在飞书 / Telegram / WhatsApp 里」，把对话接到你常用的聊天软件。

---

## 本节要点

- **安装**：用官方一键脚本（`curl ... | bash` 或 PowerShell 的 `iwr ... | iex`），脚本会在支持的系统上顺带处理 Node。
- **配置**：执行 `openclaw onboard --install-daemon`，跟着向导完成模型、Gateway、工作区等；渠道可先全部跳过。
- **确认运行**：`openclaw gateway status` 显示 Gateway 在跑即可。
- **第一次对话**：执行 `openclaw dashboard` 或浏览器打开 `http://127.0.0.1:18789/`，在 Control UI 里和助手聊天；不接 IM 也能用。

---

## 常见问题

**Q：Windows 上怎么装？**  
官方建议在 **WSL2**（Windows 自带的 Linux 子系统）里安装和运行 OpenClaw，再用浏览器访问 `http://127.0.0.1:18789`。可先安装 WSL2，在 WSL 终端里执行 macOS/Linux 的那条 `curl` 安装命令。详见 [官方文档 - Windows](https://docs.openclaw.ai/install) 或 [多语言站 www.openclawx.cloud](https://www.openclawx.cloud)。

**Q：执行 `openclaw gateway status` 说没在运行怎么办？**  
可以先手动启动一次：在终端执行 `openclaw gateway --port 18789`，保持终端不关，再在浏览器打开 `http://127.0.0.1:18789/`。若这样能正常用，说明是后台服务没装好或没启动，可重新执行 `openclaw onboard --install-daemon` 让向导再装一遍 daemon。

**Q：控制台提示「pairing required」或要批准设备？**  
这是安全机制：从**本机**浏览器访问 `127.0.0.1` 通常会**自动通过**。若你从另一台电脑或手机访问（例如通过局域网 IP），需要在运行 OpenClaw 的那台机器上执行 `openclaw devices list` 查看待批准设备，再用 `openclaw devices approve <requestId>` 批准。详见官方 [Control UI](https://docs.openclaw.ai/web/control-ui) / [Devices](https://docs.openclaw.ai/cli/devices)。

**Q：向导里选的「模型」是什么？Token 会花钱吗？**  
模型就是背后负责「想和说」的 AI；你选的提供商（如 OpenAI、Anthropic 等）会按**用量**计费，用量常按 **Token** 算——可以理解成「读/写你一句话用掉的字数单位」。向导里会让你填 API Key，用多少就扣多少，注意账号余额或用量限制即可。

---

## 延伸阅读

- 官方入门步骤（英文）：[Getting Started](https://docs.openclaw.ai/start/getting-started)  
- 向导详细说明（英文）：[Onboarding Wizard](https://docs.openclaw.ai/start/wizard)  
- Control UI 与设备配对（英文）：[Control UI](https://docs.openclaw.ai/web/control-ui)  
- 多语言文档（对非英文用户更友好）：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
