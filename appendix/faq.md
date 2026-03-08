---
title: 附录 A：常见问题速查
description: OpenClaw 安装失败、收不到消息、配置不生效、bindings、子 Agent、API 与渠道等常见问题集中整理，快速对号入座。
keywords: [OpenClaw, 常见问题, FAQ, 安装, 飞书, Telegram, WhatsApp, 故障排查]
---

# 附录 A：常见问题速查

卡住了、报错了、改完没生效？先来这里**对号入座**。下面按「安装与运行 → 渠道与消息 → 配置与人设 → 多 Agent → 开发」分类，每条都是教程里反复被问到的；若找不到你的情况，可以看每章末尾的「常见问题」和 [附录 B：官方文档与资源](references.md)，或到 [www.openclawx.cloud](https://www.openclawx.cloud) 查更完整的说明（对非英文用户更友好）。

---

## 安装与首次运行

**Q：我完全没接触过 Node / 命令行，能跟下来吗？**  
可以。教程会给出需要你复制粘贴的命令，在终端里执行即可；安装脚本在支持的系统上会尽量自动处理好 Node。详见 [1.1 OpenClaw 是什么](../01-getting-started/01-what-is-openclaw.md)、[1.2 十分钟内安装](../01-getting-started/02-first-chat.md)。

**Q：Windows 上怎么装？**  
官方建议在 **WSL2**（Windows 自带的 Linux 子系统）里安装和运行 OpenClaw，再用浏览器访问 `http://127.0.0.1:18789`。可先安装 WSL2，在 WSL 终端里执行与 macOS/Linux 相同的安装命令。详见 [官方文档 - Windows](https://docs.openclaw.ai/install) 或 [www.openclawx.cloud](https://www.openclawx.cloud)。

**Q：执行 `openclaw gateway status` 说没在运行怎么办？**  
先手动启动一次：在终端执行 `openclaw gateway --port 18789`，保持终端不关，再在浏览器打开 `http://127.0.0.1:18789/`。若这样能正常用，说明是后台服务没装好或没启动，可重新执行 `openclaw onboard --install-daemon` 让向导再装一遍。详见 [1.2](../01-getting-started/02-first-chat.md)、[1.5](../01-getting-started/05-daily-and-troubleshoot.md)。

**Q：控制台提示「pairing required」或要批准设备？**  
从本机浏览器访问 `127.0.0.1` 通常会**自动通过**。若从另一台电脑或手机访问（例如通过局域网 IP），需要在运行 OpenClaw 的那台机器上执行 `openclaw devices list` 查看待批准设备，再用 `openclaw devices approve <requestId>` 批准。详见官方 [Control UI](https://docs.openclaw.ai/web/control-ui) / [Devices](https://docs.openclaw.ai/cli/devices)。

**Q：从源码构建报错或依赖装不上？**  
确认 Node 版本 ≥ 22、已安装 pnpm；在仓库根目录执行 `pnpm install`，不要用 npm 或 yarn。若报原生模块相关错误，看官方 [Node](https://docs.openclaw.ai/install/node) 或 [Setup](https://docs.openclaw.ai/start/setup) 的故障排查。详见 [2.5 从源码运行](../02-advanced/05-extend-and-contribute.md)。

---

## 渠道与消息（飞书 / Telegram / WhatsApp）

**Q：配对码过期了怎么办？**  
配对码有效期约 **1 小时**，且每个渠道待处理请求有上限（默认 3 个）。过期后让对方再发一条消息会收到新码；用 `openclaw pairing list <channel>` 查看、`openclaw pairing approve <channel> <CODE>` 批准即可。详见 [1.3 接 IM](../01-getting-started/03-connect-im.md)。

**Q：飞书配置好凭证后收不到消息？**  
请确认：① Gateway 已运行；② 飞书开放平台里事件订阅选的是「长连接」且已添加 `im.message.receive_v1`；③ 应用已发布、机器人在飞书里能搜到。若用 webhook 模式，需在配置里设 `verificationToken`，并在开放平台填写对应回调地址。详见官方 [Feishu 文档](https://docs.openclaw.ai/channels/feishu)。

**Q：飞书配对失败或消息路由不到助手？**  
部分版本可能出现**内置飞书插件**与**手动安装的飞书插件**同时存在，导致 binding 无法正确路由、配对异常。若配置都对但消息/配对异常，请检查插件目录（如 `~/.openclaw/plugins`）是否存在重复的飞书相关插件；**删除其中一份**（保留其一即可），重启 Gateway 后再试。详见 [1.3 常见问题](../01-getting-started/03-connect-im.md)。

**Q：Telegram 群聊里 @ 机器人没反应？**  
检查 BotFather 里是否开启了群组权限（如 `/setjoingroups`）；群内是否设置了只有 @ 机器人才回复。若希望机器人看全群消息，需在 Telegram 里把机器人的「Privacy Mode」关掉（对 @BotFather 发 `/setprivacy`），并在群设置里给机器人相应权限。详见 [1.3](../01-getting-started/03-connect-im.md)。

**Q：WhatsApp 扫码后断线或收不到消息？**  
确保运行 Gateway 的电脑网络稳定；同一号码不要在多个地方同时登录 WhatsApp Web。若换设备或重装，需要重新执行 `openclaw channels login --channel whatsapp` 再扫一次码。详见 [1.3](../01-getting-started/03-connect-im.md)。

**Q：钉钉能用吗？**  
当前官方文档里没有单独写钉钉的对接方式，请以 [官方渠道列表](https://docs.openclaw.ai/channels) 或 [www.openclawx.cloud](https://www.openclawx.cloud) 为准；若后续支持，会在渠道文档中说明。详见 [附录 D 渠道支持说明](channels.md)。

**Q：想确认渠道到底连没连上？**  
执行 `openclaw channels status --probe`，会逐个检查已配置的渠道连接状态；若有报错，再结合 `openclaw logs --follow` 和该渠道官方文档的 Troubleshooting 排查。详见 [1.5](../01-getting-started/05-daily-and-troubleshoot.md)。

---

## 配置、模型与认证

**Q：改完配置没生效？**  
多数配置支持热加载，保存文件或 Control UI 保存即可。若改了端口、认证方式等，需要执行 `openclaw gateway restart`。若用了 **OPENCLAW_CONFIG_PATH** 或 **OPENCLAW_PROFILE**，确认当前进程读的是你改的那份配置。详见 [2.1 配置文件](../02-advanced/01-config-and-model.md)。

**Q：提示 "Model is not allowed" 或选了某模型没反应？**  
说明你配置了 **agents.defaults.models**（模型白名单），当前选的模型不在名单里。解决：把该模型加进 **models**，或删掉 **models** 取消白名单；可用 `/model list` 看当前允许的列表。详见 [2.1](../02-advanced/01-config-and-model.md)。

**Q：API Key 写在哪？**  
不要写在 `openclaw.json` 里。用环境变量（如 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY`）或通过 `openclaw onboard` / `openclaw configure` 写入 OpenClaw 的认证存储（在状态目录下），配置里只写模型名（provider/model）。详见 [2.1](../02-advanced/01-config-and-model.md)。

**Q：想在同一台机器跑两套互不影响的 OpenClaw？**  
用不同的 **OPENCLAW_CONFIG_PATH** 和 **OPENCLAW_STATE_DIR** 启动两个 Gateway，并指定不同端口（如 `--port 18789` 和 `--port 19001`）。详见官方 [Multiple gateways](https://docs.openclaw.ai/gateway/multiple-gateways)。

**Q：向导里选的「模型」是什么？Token 会花钱吗？**  
模型就是背后负责「想和说」的 AI；你选的提供商（如 OpenAI、Anthropic）会按**用量**计费，用量常按 **Token** 算——可以理解成「读/写你一句话用掉的字数单位」。用多少就扣多少，注意账号余额或用量限制即可。详见 [1.2](../01-getting-started/02-first-chat.md)、[附录 C 术语简表](glossary.md)。

---

## 工作区、人设与记忆

**Q：我没有 AGENTS.md 或 SOUL.md，怎么办？**  
在终端执行 `openclaw setup`，会在工作区里创建缺失的默认文件，不会覆盖已有文件。若希望完全自己管理，可在配置里设 `agent.skipBootstrap: true` 关闭自动创建。详见 [1.4 人设与记忆](../01-getting-started/04-make-it-yours.md)。

**Q：改了 AGENTS.md / SOUL.md 但助手好像没按新规矩来？**  
引导文件只在**新会话的第一轮**注入；若是在**已有会话**里继续聊，当前会话已经用过旧内容了。**新开一个会话**（新对话或刷新后重聊）才会读到最新文件。详见 [2.3 Agent 与工作区](../02-advanced/03-agent-and-workspace-deep.md)。

**Q：TOOLS.md 里写了某工具，为什么助手还是不用？**  
TOOLS.md 只是**给助手看的说明**，不控制「能不能用」某个工具。工具是否可用由配置和权限决定；若该工具在配置里被 **deny** 或在 profile 里没放行，助手根本拿不到，自然不会用。先确认配置里该工具已允许。详见 [1.4](../01-getting-started/04-make-it-yours.md)、[2.4 工具与 Skills](../02-advanced/04-tools-and-skills.md)。

**Q：MEMORY.md 和 memory/ 里的文件有什么区别？**  
MEMORY.md 适合**长期、精选**的重要信息（偏好、决定、关键事实），且只在你的主会话里加载。memory/YYYY-MM-DD.md 是**按天的流水账**，助手会读「今天 + 昨天」来延续近期上下文；群聊等场景不会读 MEMORY.md。详见 [1.4](../01-getting-started/04-make-it-yours.md)。

**Q：工作区默认路径能改吗？**  
可以。在 `~/.openclaw/openclaw.json` 里设置 `agent.workspace` 为你想要的路径（支持 `~`）。改完后若新路径下还没有那些 .md 文件，可执行 `openclaw setup --workspace <路径>` 生成默认文件。详见 [1.4](../01-getting-started/04-make-it-yours.md)。

**Q：队列模式在哪儿设？沙箱里助手找不到工作区文件？**  
队列模式：全局在配置里 `messages.queue.mode` 和 `messages.queue.byChannel.<渠道>`；当前会话也可用 `/queue collect`、`/queue steer` 等（取决于渠道是否支持）。沙箱：若 `workspaceAccess` 是 `none`，沙箱里没有挂你的工作区；需要读工作区时改为 `ro` 或 `rw`。详见 [2.3](../02-advanced/03-agent-and-workspace-deep.md)。

---

## Gateway、API 与控制台

**Q：开了 chatCompletions 但请求 404？**  
确认配置里 `gateway.http.endpoints.chatCompletions.enabled` 为 `true` 且已生效（改配置后若未热加载，需 `openclaw gateway restart`）；请求 URL 是否为 `http://<host>:<port>/v1/chat/completions`（端口与 Gateway 一致）。详见 [2.2 Gateway 与 API](../02-advanced/02-gateway-and-api.md)。

**Q：API 返回 401 / 403？**  
检查请求头是否带 `Authorization: Bearer <token>`，且 token 与当前 Gateway 的 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）一致；若 Gateway 用的是 password 模式，则用密码而非 token。详见 [2.2](../02-advanced/02-gateway-and-api.md)。

**Q：从外网如何安全访问家里的 Gateway？**  
不要直接把 18789 映射到公网。推荐：本机用 **SSH 隧道**连回家中机器，或家中机器与常用电脑都装 **Tailscale**，用 Tailscale 地址访问；若需对外提供控制台，用 Tailscale Serve 等限定在可信网络。详见官方 [Remote access](https://docs.openclaw.ai/gateway/remote)。

**Q：更新后控制台或渠道异常？**  
先执行 `openclaw gateway restart`。若仍异常，跑一遍 `openclaw doctor`，按提示做修复或迁移；再查官方 [Updating](https://docs.openclaw.ai/install/updating) 和对应渠道的 Troubleshooting。详见 [1.5](../01-getting-started/05-daily-and-troubleshoot.md)。

**Q：日志太多、不知道从哪看起？**  
用 `openclaw logs --follow` 实时看，再在另一个终端复现问题（例如发一条消息），看同一时刻日志里出现的报错或警告；把那段贴到搜索引擎或官方文档里搜，往往能定位到原因。详见 [1.5](../01-getting-started/05-daily-and-troubleshoot.md)。

---

## 工具与 Skills

**Q：Skill 改完了没反应？**  
确认 SKILL.md 在正确目录（工作区 `skills/技能名/SKILL.md` 或 `~/.openclaw/skills/技能名/SKILL.md`），且**新开了一个会话**（或重启了 Gateway）；同一会话内可能已经加载过旧版本。详见 [2.4](../02-advanced/04-tools-and-skills.md)。

**Q：想禁止助手执行任何命令？**  
在配置里设 `tools.deny: ["group:runtime"]`（或 `["exec", "bash", "process"]`）；若用 profile，可选 `tools.profile: "messaging"` 再按需 allow 个别工具。详见 [2.4](../02-advanced/04-tools-and-skills.md)。

**Q：第三方 Skill 安全吗？**  
要当**不可信**处理：先看 SKILL.md 里有没有调 exec、写文件、发消息等；再考虑用沙箱、或给该 Agent 用更小的 tools.profile。不要把密钥写进 Skill 正文；用环境变量或配置 SecretRef。详见官方 [Security](https://docs.openclaw.ai/gateway/security)、[2.4](../02-advanced/04-tools-and-skills.md)。

---

## 多 Agent 与子 Agent

**Q：bindings 写了但消息还是进了 main？**  
检查 match 是否写对：channel、accountId（若用多账号）、peer（若按人分）。用 `openclaw agents list --bindings` 看当前绑定是否生效；**最具体**的规则优先，若有多条匹配，按配置里顺序取第一条。详见 [3.1 多 Agent 搭建](../03-practice/01-multi-agent-setup.md)。

**Q：多个 Agent 能共用一个工作区吗？**  
不推荐。每个 Agent 应有**独立 workspace**，否则会话、人设、记忆会混在一起。若想「共享一部分设定」，可以复制 AGENTS.md/SOUL.md 到多个工作区再各自改，或把公共内容放到共享 Skill（`~/.openclaw/skills`）。详见 [3.1](../03-practice/01-multi-agent-setup.md)。

**Q：飞书 / Telegram / WhatsApp 的 bindings 怎么写？**  
在 bindings 里写 `match: { channel: "feishu" }` 或 `"telegram"`、`"whatsapp"`；若该渠道配了多个 account，用 `accountId: "xxx"` 指定；若按发送者分，用 `peer: { kind: "direct", id: "..." }`。完整示例见官方 [Multi-Agent](https://docs.openclaw.ai/concepts/multi-agent)。详见 [3.1](../03-practice/01-multi-agent-setup.md)。

**Q：派出去子 Agent 后没看到汇报？**  
汇报是**尽力而为**的：若 Gateway 中途重启或异常，未完成的 announce 可能丢失。正常情况下跑完会有一条结果消息；可用 `/subagents list` 看状态，用 `/subagents log <id>` 看该次输出。详见 [3.2 子 Agent 与并行任务](../03-practice/02-subagents-and-tasks.md)。

**Q：能派到「法律」「会计」那种别的 Agent 吗？**  
可以，但要先在 **agents.list** 里配好那个 Agent（见 3.1），再在该主助手的配置里设 **subagents.allowAgents**，把那个 Agent 的 id 加进去（或 `["*"]` 允许所有）；spawn 时写那个 agentId 即可。详见 [3.2](../03-practice/02-subagents-and-tasks.md)、[3.3 律师+会计师](../03-practice/03-case-multi-role-legal.md)、[3.4 产品+开发+测试](../03-practice/04-case-multi-role-dev.md)。

**Q：主助手没有自动派 legal/accountant 或 product/coding/qa，而是自己答了？**  
确认 main（或你对话所在 Agent）的配置里 **subagents.allowAgents** 已包含对应 Agent id，且已重启 Gateway；再确认你对主助手下的指令里明确说了「派某某 Agent」或「用 sessions_spawn 让某某回答」。若主助手没有 sessions_spawn 权限或没被提示用子 Agent，它可能直接自己答。详见 [3.3](../03-practice/03-case-multi-role-legal.md)、[3.4](../03-practice/04-case-multi-role-dev.md)。

**Q：可以子 Agent 再派子 Agent 吗？**  
可以，但需在配置里把 **agents.defaults.subagents.maxSpawnDepth** 设为 2 或更大（默认 1）；深度 2 时适合「主助手 → 协调子 Agent → 多个干活子 Agent」的模式。详见官方 [Sub-Agents](https://docs.openclaw.ai/tools/subagents)。详见 [3.2](../03-practice/02-subagents-and-tasks.md)。

---

## 开发与贡献

**Q：link 之后 `openclaw` 还是旧版本？**  
确认当前 shell 的 `which openclaw` 指向的是 link 的目标（例如 pnpm 的 global bin 目录）；必要时先 `pnpm unlink --global` 再重新 `pnpm link --global`，或直接用 `pnpm openclaw` 从仓库运行。详见 [2.5](../02-advanced/05-extend-and-contribute.md)。

**Q：想给 OpenClaw 提 PR，从哪开始？**  
在 GitHub 上 fork [openclaw/openclaw](https://github.com/openclaw/openclaw)，clone 你的 fork，按 2.5 从源码构建；改完在 fork 里提交，再在 GitHub 上对原仓库提 Pull Request。具体规范看仓库的 CONTRIBUTING 和 README。详见 [2.5](../02-advanced/05-extend-and-contribute.md)。

**Q：插件写好了怎么装？**  
若插件是本地目录：`openclaw plugins install /path/to/your-plugin`（若支持）；若是 npm 包：`openclaw plugins install 包名`。装好后在配置里启用并填 `plugins.entries.<id>.config`，重启 Gateway。详见官方 [Plugins](https://docs.openclaw.ai/tools/plugin)、[2.5](../02-advanced/05-extend-and-contribute.md)。

---

## 本节要点

- 本附录把教程各章里的**常见问题**按主题集中在一起，方便你「卡住了先对号入座」。
- 分类大致为：**安装与运行**、**渠道与消息**、**配置与认证**、**工作区与人设**、**Gateway 与 API**、**工具与 Skills**、**多 Agent 与子 Agent**、**开发与贡献**。
- 每条都给出了简短解决办法，并指向对应章节或官方文档；更完整、最新的说明以 [官方文档](https://docs.openclaw.ai) 和 [www.openclawx.cloud](https://www.openclawx.cloud) 为准。

---

## 延伸阅读

- [附录 B：官方文档与资源](references.md) — 官方文档、多语言站、API、CLI、社区链接  
- [附录 C：术语简表](glossary.md) — Token、Agent、Gateway 等词一句话解释  
- [附录 D：渠道支持说明](channels.md) — 飞书、Telegram、WhatsApp 的文档依据  
- [1.5 日常怎么用、出问题怎么查](../01-getting-started/05-daily-and-troubleshoot.md) — 日志、重启、更新与排查思路  

[← 返回目录](../README.md)
