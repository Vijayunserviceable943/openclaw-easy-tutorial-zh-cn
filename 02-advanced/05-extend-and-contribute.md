---
title: 从源码运行与参与贡献
description: OpenClaw 从源码克隆、pnpm 构建与运行、插件安装与开发、PR 与贡献、问题反馈与社区入口。
keywords: [OpenClaw, 源码, 构建, 插件, Plugins, 贡献, PR]
---

# 从源码运行与参与贡献

读完这一节，你会知道如何**从源码**克隆、构建并运行 OpenClaw（方便改代码、调试或参与开发），**插件**大致怎么装、怎么扩展，以及**问题反馈、安全报告、社区**该找哪里。适合想跑最新代码、写插件或给项目做贡献的你。

---

## 为什么要从源码跑

通常用安装脚本或 `npm install -g openclaw` 就够了。从**源码**跑适合：

- 你想改 OpenClaw 本身或看代码逻辑；
- 你想用**尚未正式发版**的最新改动（main 分支）；
- 你在给 OpenClaw 做**贡献**（修 bug、加功能），需要本地构建验证。

从源码跑需要本机有 **Node 22+** 和 **pnpm**（官方仓库用 pnpm 管理依赖）。

---

## 从源码构建与运行

### 1. 克隆仓库并安装依赖

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
```

若你已有 clone，更新后重新装依赖、构建：

```bash
cd openclaw
git pull
pnpm install
```

### 2. 构建

```bash
pnpm ui:build
pnpm build
```

`ui:build` 会构建 Control UI 等前端资源；`build` 会打包 CLI 和 Gateway。构建完成后，可执行文件在仓库里（例如通过 `node openclaw.mjs` 或下面 link 后的 `openclaw` 命令调用）。

### 3. 让本机能用 `openclaw` 命令（二选一）

**方式 A：全局 link（推荐，方便日常用）**

```bash
pnpm link --global
```

之后在任意目录都能直接打 `openclaw`，用的就是你刚构建的版本。

**方式 B：不 link，在仓库里用 pnpm 调**

```bash
pnpm openclaw gateway --port 18789
pnpm openclaw onboard --install-daemon
```

适合「只在这台机器上临时跑一份从源码来的」场景。

### 4. 跑 Gateway 与配置

- 若还没做过配置，执行一次向导：`openclaw onboard --install-daemon`（若已 link）或 `pnpm openclaw onboard --install-daemon`。
- 直接跑 Gateway（不装成系统服务）：  
  `openclaw gateway --port 18789`  
  或从仓库里：  
  `node openclaw.mjs gateway --port 18789 --verbose`  
  开发时可用 `pnpm gateway:watch` 等（见官方 [Setup](/start/setup)）。

你的**配置和工作区**仍然在 `~/.openclaw/`（或你设的 OPENCLAW_STATE_DIR），和用 npm 装的一样；只是「执行文件」换成了你构建的这份。

---

## 插件：扩展能力与开发入口

OpenClaw 用**插件**（Plugin）扩展能力：新渠道、新工具、新 CLI 命令等可以做成插件，按需安装，不塞进主仓库。

- **安装官方插件**（例如飞书、Voice Call）：  
  `openclaw plugins install @openclaw/feishu`  
  装好后在配置里 `plugins.entries.<id>.config` 里填参数，重启 Gateway。
- **看已加载的插件**：  
  `openclaw plugins list`

插件是** TypeScript 模块**，由 Gateway 进程加载；可以注册：新工具、Gateway RPC、HTTP 路由、CLI 子命令、Skills 等。若你要**自己写插件**（新渠道、新工具），需要按官方 [Plugins](https://docs.openclaw.ai/tools/plugin)、[Plugin manifest](https://docs.openclaw.ai/plugins/manifest)、[Plugin agent tools](https://docs.openclaw.ai/plugins/agent-tools) 的约定来写，并装到 `~/.openclaw/extensions/` 或通过 `openclaw plugins install <path>` 安装。插件和主进程同进程运行，视为**可信代码**，注意安全。

渠道（飞书、Telegram、WhatsApp 等）中，部分以**插件**形式提供（如飞书 `@openclaw/feishu`）；装好插件、配好凭证后，该渠道才会在 Gateway 里可用。详见各渠道的官方文档。

---

## 问题反馈与社区

- **Bug、功能建议、使用问题**：到 OpenClaw 官方仓库的 **Issues** 提：[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues)。提之前可以先搜一下是否已有类似 issue；描述清楚环境、复现步骤和期望行为，便于维护者处理。
- **安全漏洞**：请**不要**在公开 Issue 里贴漏洞细节。按官方 [Security](https://docs.openclaw.ai/gateway/security) 说明，通过 **security@openclaw.ai** 负责任地报告；仓库根目录的 **SECURITY.md** 里也有流程说明。
- **文档与帮助**：  
  - 英文：[https://docs.openclaw.ai](https://docs.openclaw.ai)  
  - 多语言（含中文）：[https://www.openclawx.cloud](https://www.openclawx.cloud)  
- **关于项目与致谢**：贡献者、许可证（MIT）等见官方 [Credits](https://docs.openclaw.ai/reference/credits)。

参与贡献（修 bug、提 PR）一般流程：fork 仓库 → 在本地从源码构建、改代码、测试 → 在 GitHub 上提 Pull Request；具体约定以仓库的 CONTRIBUTING、README 和 issue 模板为准。

---

## 本节要点

- **从源码跑**：克隆 [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)，`pnpm install` → `pnpm ui:build` → `pnpm build`，再用 `pnpm link --global` 或 `pnpm openclaw` 运行；配置与工作区仍在 `~/.openclaw/`。
- **插件**：用 `openclaw plugins install <包名>` 安装；可扩展工具、渠道、CLI 等；自研插件需按官方 Plugin 文档实现并安装；飞书等渠道以插件形式提供。
- **反馈**：一般问题到 GitHub Issues；安全漏洞发 **security@openclaw.ai**；文档与多语言站见 docs.openclaw.ai、www.openclawx.cloud；贡献流程以仓库说明为准。

---

## 常见问题

**Q：从源码构建报错或依赖装不上？**  
确认 Node 版本 ≥ 22、已安装 pnpm；在仓库根目录执行 `pnpm install`，不要用 npm 或 yarn。若报原生模块相关错误，看官方 [Node](/install/node) 或 [Setup](/start/setup) 的故障排查。

**Q：link 之后 `openclaw` 还是旧版本？**  
确认当前 shell 的 `which openclaw` 指向的是 link 的目标（例如 pnpm 的 global bin 目录）；必要时先 `pnpm unlink --global` 再重新 `pnpm link --global`，或直接用 `pnpm openclaw` 从仓库运行。

**Q：想给 OpenClaw 提 PR，从哪开始？**  
在 GitHub 上 fork [openclaw/openclaw](https://github.com/openclaw/openclaw)， clone 你的 fork，按上面步骤从源码构建；改完在 fork 里提交，再在 GitHub 上对原仓库提 Pull Request。具体规范看仓库的 CONTRIBUTING 和 README。

**Q：插件写好了怎么装？**  
若插件是本地目录：`openclaw plugins install /path/to/your-plugin`（若支持）；若是 npm 包：`openclaw plugins install 包名`。装好后在配置里启用并填 `plugins.entries.<id>.config`，重启 Gateway。详见 [Plugins](https://docs.openclaw.ai/tools/plugin)。

---

## 延伸阅读

- 安装方式总览（含 From source）（英文）：[Install](https://docs.openclaw.ai/install)  
- 开发者本地跑法与工作流（英文）：[Setup](https://docs.openclaw.ai/start/setup)  
- 插件安装与开发（英文）：[Plugins](https://docs.openclaw.ai/tools/plugin)、[Plugin manifest](https://docs.openclaw.ai/plugins/manifest)  
- 安全与漏洞报告（英文）：[Security](https://docs.openclaw.ai/gateway/security)  
- 项目致谢与许可证（英文）：[Credits](https://docs.openclaw.ai/reference/credits)  
- 官方仓库：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
