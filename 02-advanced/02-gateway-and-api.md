---
title: Gateway 是啥？怎么用 API 和别的系统对接？
description: OpenClaw Gateway 在链路中的角色，Control UI 与远程访问，HTTP API chat completions、Bearer Token、SSH 隧道与 Tailscale。
keywords: [OpenClaw, Gateway, API, chat completions, 远程访问, Control UI]
---

# Gateway 是啥？怎么用 API 和别的系统对接？

读完这一节，你会把 **Gateway** 在整条链路里的角色想清楚，知道怎么在**本机或远程**打开控制台、怎么用 **HTTP API** 让别的程序调用你的助手，以及远程访问时的大致做法（SSH 隧道、Tailscale）。适合已经日常在用 OpenClaw、想和自建系统或脚本对接的你。

---

## Gateway 在整条链路里干啥

前面我们把 Gateway 比作「桥」：飞书、Telegram、WhatsApp 发来的消息先到它，再由它转给助手；助手的回复也经它发回对应渠道。更准确一点说：

- **Gateway** 是一个**常驻进程**（你执行 `openclaw gateway` 启动的就是它），同时提供 **WebSocket** 和 **HTTP**，默认端口 **18789**。
- 它负责：接收各**渠道**（Channel）的消息、维护**会话**（Session）、把请求交给**助手**（Agent）跑、把回复发回渠道；控制台（Control UI）和节点设备也通过它连上来。
- 所以：**会话、路由、渠道连接**都以 Gateway 为「唯一真相来源」；只跑一个 Gateway 进程（除非你刻意用多套配置多开），飞书/Telegram/WhatsApp 都连到同一个 Gateway。

你用自己的程序或脚本和 OpenClaw「对接」，本质上就是：和这台 Gateway **对话**——要么用网页（Control UI），要么用 **HTTP API** 或 WebSocket 协议。

---

## 控制台与远程访问

**Control UI**（控制台）是 Gateway 自带的网页界面：聊天、看会话、改配置等。它和 Gateway 在**同一个端口**上，本机访问就是：

- 浏览器打开：`http://127.0.0.1:18789/`（或 `http://localhost:18789/`）
- 或执行：`openclaw dashboard`

第一次从**本机**浏览器连，一般会自动通过；从**别的电脑**连（例如你在公司想访问家里的 Gateway），就需要「能访问到那台机器」且做**设备配对**（在运行 Gateway 的机器上执行 `openclaw devices list`、`openclaw devices approve <requestId>`）。  
要安全地「从外面」访问，通常有两种方式：

1. **SSH 隧道**  
   在你当前电脑上执行：  
   `ssh -N -L 18789:127.0.0.1:18789 用户@运行Gateway的机器`  
   这样本机的 18789 会转发到那台机器的 18789；然后你在本机浏览器打开 `http://127.0.0.1:18789/`，实际连的就是远程的 Gateway。隧道保持期间，用起来和本地一样。

2. **Tailscale 等 VPN / 内网**  
   若运行 Gateway 的机器和你的电脑都在同一 Tailscale 网络里，可以用 Tailscale 的地址直接访问（例如 `http://那台机器的 tailscale 名:18789`）。Gateway 还可配合 **Tailscale Serve** 暴露控制台，详见官方 [Remote access](https://docs.openclaw.ai/gateway/remote)、[Tailscale](https://docs.openclaw.ai/gateway/tailscale)。

**安全提醒**：不要把 Gateway 端口直接暴露到公网。用隧道或 Tailscale 等限制在可信网络内；若必须对外，务必配好认证（token/密码）并理解下面 API 的「全权限」性质。

---

## 用 HTTP API 和别的系统对接

Gateway 提供一个 **OpenAI 兼容**的 HTTP 接口，方便你自己的程序、脚本或第三方工具（如支持 OpenAI 兼容接口的客户端）直接发请求、拿助手回复。这样就能实现「别的系统调你的 OpenClaw 助手」。

### 接口是什么、在哪

- **路径**：`POST /v1/chat/completions`
- **地址**：和 Gateway 同端口，例如 `http://<Gateway 所在机器>:18789/v1/chat/completions`
- 该接口**默认关闭**，需要在配置里打开（见下）。

请求会走和「控制台 / 渠道」同一套助手逻辑（同一套路由、权限、模型），所以行为一致。

### 认证

使用 Gateway 的认证方式，在请求头里带 **Bearer Token**：

- `Authorization: Bearer <token>`
- Token 就是你配置里的 `gateway.auth.token`（或环境变量 `OPENCLAW_GATEWAY_TOKEN`）。若用的是密码模式，则用 `gateway.auth.password` / `OPENCLAW_GATEWAY_PASSWORD`。

认证失败过多时，可能返回 429（含 `Retry-After`），需控制重试。

### 安全边界（必读）

这个 HTTP 接口是**全操作员权限**：谁拿着合法 token/密码，谁就被视为「这台 Gateway 的信任操作者」。

- 没有「按用户缩小权限」的模型；一旦通过认证，请求会以完整助手能力执行（包括工具等），和你在控制台里操作等价。
- 因此：**不要**把该接口直接暴露到公网；只在内网、loopback 或 Tailscale 等可信入口使用，并妥善保管 token/密码。详见官方 [Security](https://docs.openclaw.ai/gateway/security)、[Remote access](https://docs.openclaw.ai/gateway/remote)。

### 在配置里打开接口

在 `~/.openclaw/openclaw.json` 里加上：

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

保存后若未自动生效，执行 `openclaw gateway restart`。

### 选哪个助手（多 Agent 时）

若你配了多个 Agent，可以在请求里指定用哪一个：

- 在请求体里用 **model** 字段：`"model": "openclaw:main"` 或 `"model": "openclaw:<agentId>"`（如 `openclaw:beta`）。
- 或用请求头：`x-openclaw-agent-id: main`（默认是 `main`）。

高级用法还可用 `x-openclaw-session-key` 指定会话，详见官方 [OpenAI Chat Completions](https://docs.openclaw.ai/gateway/openai-http-api)。

### 简单示例（curl）

**不流式**：

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer 你的GatewayToken' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"你好"}]
  }'
```

**流式**（SSE）：在请求体里加 `"stream": true`，用 `curl -N` 接收事件流。详见官方文档示例。

---

## 用 CLI 发消息到渠道（脚本/自动化）

若你只是想「从脚本里往飞书/Telegram/WhatsApp 发一条消息」（由助手回复或仅发送），可以用 **CLI**，无需自己调 HTTP：

```bash
openclaw message send --channel telegram --target @某个用户 --message "需要提醒的内容"
```

`--channel` 填 `telegram`、`whatsapp`、`feishu` 等；`--target` 格式因渠道而异（见官方 [message](https://docs.openclaw.ai/cli/message)）。这样可以在 cron、CI 或自己的脚本里调用，实现简单自动化。更复杂的自动化（如 webhook、定时任务）可看官方 [Automation](https://docs.openclaw.ai/automation)、[Webhooks](https://docs.openclaw.ai/cli/webhooks) 等文档。

---

## 本节要点

- **Gateway** 是唯一常驻进程，负责渠道、会话、助手调度；Control UI 和 HTTP API 都在同一端口（默认 18789）。
- **控制台**：本机 `http://127.0.0.1:18789/` 或 `openclaw dashboard`；远程用 **SSH 隧道**或 **Tailscale** 等访问，并做设备配对。
- **HTTP API**：`POST /v1/chat/completions`，OpenAI 兼容；默认关闭，需在配置里 `gateway.http.endpoints.chatCompletions.enabled: true`；认证用 Bearer Token；视为**全操作员权限**，仅限内网/可信网络。
- 多 Agent 时用 `model: "openclaw:<agentId>"` 或 `x-openclaw-agent-id` 指定助手。
- 脚本/自动化可先用 **openclaw message send** 发渠道消息；更复杂场景查官方 Automation、Webhooks 文档。

---

## 常见问题

**Q：开了 chatCompletions 但请求 404？**  
确认配置里 `gateway.http.endpoints.chatCompletions.enabled` 为 `true` 且已生效（改配置后若未热加载，需 `openclaw gateway restart`）；请求 URL 是否为 `http://<host>:<port>/v1/chat/completions`（端口与 Gateway 一致）。

**Q：返回 401 / 403？**  
检查请求头是否带 `Authorization: Bearer <token>`，且 token 与当前 Gateway 的 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）一致；若 Gateway 用的是 password 模式，则用密码而非 token。

**Q：从外网如何安全访问家里的 Gateway？**  
不要直接把 18789 映射到公网。推荐：本机用 **SSH 隧道**连回家中机器，或家中机器与常用电脑都装 **Tailscale**，用 Tailscale 地址访问；若需对外提供控制台，用 Tailscale Serve 等限定在可信网络。详见 [Remote access](https://docs.openclaw.ai/gateway/remote)。

**Q：API 和 Control UI 用的是同一套助手吗？**  
是。请求都走同一套 Gateway 逻辑，同一模型、同一工作区、同一套权限；只是入口不同（网页 vs HTTP）。

---

## 延伸阅读

- Gateway 总览（英文）：[Gateway](https://docs.openclaw.ai/gateway)  
- OpenAI 兼容接口（英文）：[OpenAI Chat Completions](https://docs.openclaw.ai/gateway/openai-http-api)  
- 远程访问（英文）：[Remote access](https://docs.openclaw.ai/gateway/remote)  
- 安全（英文）：[Security](https://docs.openclaw.ai/gateway/security)  
- 发送消息 CLI（英文）：[message](https://docs.openclaw.ai/cli/message)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
