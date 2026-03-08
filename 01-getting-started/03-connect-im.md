---
title: 让助手出现在飞书 / Telegram / WhatsApp 里
description: 用最少几步把 OpenClaw 助手接到飞书、Telegram 或 WhatsApp，在 IM 里直接对话。配对、长连接、webhook、安全权限设置。
keywords: [OpenClaw, 飞书, Telegram, WhatsApp, 配对, pairing, 渠道, Channel]
---

# 让助手出现在飞书 / Telegram / WhatsApp 里

读完这一节，你可以从**飞书、Telegram、WhatsApp**里选一个你最常用的，用**最少几步**把助手接上去，在对应 App 里直接和助手对话。同时你会知道怎样设置「谁可以给助手发消息」，避免对陌生人敞开大门。

---

## 先想清楚一件事：安全

把助手接到聊天软件后，**发到那个渠道的消息都会变成助手的输入**。若不做限制，任何人都可能给你的助手发消息、占用你的模型和配额。所以官方建议：**一开始就收紧权限**。

- **飞书 / Telegram**：用**配对（pairing）**或**白名单（allowFrom）**。配对的意思是：陌生人第一次发消息时，助手只回复一个短期有效的「配对码」，不处理内容；你在自己电脑上执行命令批准后，这个人才算「允许名单」里的人。
- **WhatsApp**：尽量用**单独一个手机号**给助手用，不要用你日常私人的主号；并在配置里写上 **allowFrom**（允许给助手发消息的号码列表），不要对全网开放。

下面按渠道分别说**最少步骤**；你只需先选一个渠道打通即可。

---

## 渠道一：飞书（适合国内团队）

飞书需要先安装 OpenClaw 的**飞书插件**，再在飞书开放平台创建一个应用，把应用凭证填进 OpenClaw。

### 1. 安装飞书插件

在终端执行：

```bash
openclaw plugins install @openclaw/feishu
```

> **注意**：若你使用的版本（约 2026.2 起）同时带有内置飞书插件，可能出现「内置 + 自装」两份插件并存，导致消息路由或配对异常。若配对/收不到消息且配置无误，请到本节末尾「常见问题」查看「飞书配对失败或消息路由不到助手」的解决办法（删除重复的插件目录之一）。

### 2. 在飞书开放平台创建应用并拿到凭证

1. 打开 [飞书开放平台](https://open.feishu.cn/app) 并登录（国际版 Lark 用 [open.larksuite.com](https://open.larksuite.com/app)，后面配置里要把 `domain` 设为 `lark`）。
2. 创建企业自建应用，填好名称、描述等。
3. 在「凭证与基础信息」里复制 **App ID**（形如 `cli_xxx`）和 **App Secret**，妥善保管。
4. 在「权限」里按官方文档要求开通机器人所需权限（如收发消息、获取用户信息等）；飞书文档里有一份可批量导入的权限列表，见 [Feishu 官方文档](https://docs.openclaw.ai/channels/feishu)。
5. 在「应用能力」里启用**机器人**。
6. 在「事件订阅」里选择**使用长连接接收事件**（WebSocket），并添加事件 `im.message.receive_v1`。  
   **注意**：这一步前请先确保 OpenClaw 的 Gateway 已在运行（`openclaw gateway status`），否则长连接可能保存失败。
7. 在「版本管理与发布」里创建版本、提交审核并发布（企业自建应用通常很快通过）。

### 3. 在 OpenClaw 里配置飞书

在终端执行：

```bash
openclaw channels add
```

选择 **Feishu**，按提示填入刚才的 App ID 和 App Secret。若你之前没跑过向导，也可以先执行 `openclaw onboard`，在向导里选 Feishu 并填入凭证。

### 4. 重启 Gateway 并在飞书里试一下

执行 `openclaw gateway restart`（若用后台服务）或保持 `openclaw gateway --port 18789` 运行。在飞书里找到你的机器人，发一条消息，助手应能回复。若你希望**只允许特定人**和机器人私聊，可在配置里设置 **allowFrom**（飞书侧是用户 open_id 列表）或使用 **pairing**（陌生人先拿配对码，你在本机执行 `openclaw pairing list feishu` / `openclaw pairing approve feishu <CODE>` 批准）。详见官方 [Feishu 渠道文档](https://docs.openclaw.ai/channels/feishu)。

---

## 渠道二：Telegram

Telegram 不需要扫码，只要在 Telegram 里用 **@BotFather** 创建一个机器人，拿到 **Bot Token**，填进 OpenClaw 并启动 Gateway 即可。默认用**配对**：陌生人私聊机器人会收到一个配对码，你在本机批准后才会处理他的消息。

### 1. 在 Telegram 里创建机器人并拿到 Token

在 Telegram 里搜索 **@BotFather**（认准官方），对话里发送 `/newbot`，按提示起名、选用户名，完成后 BotFather 会给你一串 **Bot Token**（形如 `123456:ABC-DEF...`），复制保存。

### 2. 在 OpenClaw 里配置 Telegram

编辑配置文件 `~/.openclaw/openclaw.json`（若没有，可先执行 `openclaw onboard` 让向导生成一个），在 `channels` 里加上 `telegram`，例如：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "这里填你的 Bot Token",
      "dmPolicy": "pairing",
      "groups": { "*": { "requireMention": true } }
    }
  }
}
```

- **dmPolicy: "pairing"**：私聊默认走配对，陌生人先拿码，你批准后才加入允许名单。
- **groups** 里 `requireMention: true` 表示群里只有 @ 提到机器人时才会回复，避免刷屏。

你也可以用环境变量：`TELEGRAM_BOT_TOKEN=你的Token`，仅对默认账号生效。

### 3. 启动 Gateway 并批准第一个私聊

```bash
openclaw gateway --port 18789
```

（若已装成后台服务，用 `openclaw gateway status` 确认在跑即可。）  
用你的 Telegram 账号给刚建的机器人发一条消息，机器人会回复一个**配对码**（约 8 位，1 小时内有效）。在你运行 OpenClaw 的电脑上执行：

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <把这里的配对码填进去>
```

批准后，再发一条消息，助手就会正常回复。以后这个账号的私聊都会直接进助手，无需再次配对。

---

## 渠道三：WhatsApp

WhatsApp 通过「WhatsApp Web」方式对接：需要你在本机执行一次登录命令，用**给助手用的那个手机号**扫二维码，之后 Gateway 会保持连接。官方建议用**专用手机号**，不要用私人主号，否则所有发给你的私聊都会变成助手输入。

### 1. 先定好「谁可以给助手发消息」

在 `~/.openclaw/openclaw.json` 里为 WhatsApp 配置访问策略，例如只允许你自己的号码：

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+86你的手机号"],
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["+86你的手机号"]
    }
  }
}
```

**allowFrom** 即「允许给助手发私聊的号码列表」；**groupAllowFrom** 是允许在群里触发助手的号码。若你希望陌生人先走配对再放行，可把 `dmPolicy` 改为 `"pairing"`，再用下面的 `pairing list` / `pairing approve` 批准。

### 2. 用 WhatsApp 账号登录（扫码）

在终端执行：

```bash
openclaw channels login --channel whatsapp
```

终端会提示你用**给助手用的那个 WhatsApp 账号**（建议专用号）扫描二维码；若有多账号，可加 `--account 账号名`。扫码成功后，该账号就与 OpenClaw 绑定了。

### 3. 启动 Gateway

```bash
openclaw gateway --port 18789
```

（或确认后台服务已在运行。）用你在 **allowFrom** 里的号码给这个 WhatsApp 发消息，助手应能回复。若你用的是 **pairing**，第一次会收到配对码，在本机执行：

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <配对码>
```

即可。

---

## 钉钉能用吗？

当前官方文档里**没有单独写钉钉**的对接步骤。若你需要钉钉，请以 [官方渠道列表](https://docs.openclaw.ai/channels) 或 [多语言文档站 www.openclawx.cloud](https://www.openclawx.cloud) 为准；若后续支持，会在渠道文档中说明。

---

## 本节要点

- **安全**：别对全网开放。飞书/Telegram 用配对或白名单；WhatsApp 建议专用号 + allowFrom。
- **飞书**：安装插件 `@openclaw/feishu` → 飞书开放平台创建应用、拿 App ID/Secret、配权限与机器人、事件订阅（长连接）→ `openclaw channels add` 选 Feishu 填入 → 重启 Gateway，在飞书里找机器人发消息。
- **Telegram**：@BotFather 创建机器人拿 Token → 在配置里写 `telegram.botToken`、`dmPolicy: "pairing"` → 启动 Gateway → 私聊机器人拿配对码 → `openclaw pairing list/approve telegram <CODE>`。
- **WhatsApp**：在配置里设好 allowFrom 或 pairing → `openclaw channels login --channel whatsapp` 用专用号扫码 → 启动 Gateway，用允许的号码发消息（若用 pairing 需先 approve）。
- **钉钉**：以官方渠道列表为准，当前文档未单独说明。

---

## 常见问题

**Q：配对码过期了怎么办？**  
配对码有效期约 **1 小时**，且每个渠道待处理请求有上限（默认 3 个）。过期后让对方再发一条消息，会收到新码；你再用 `openclaw pairing list <channel>` 查看、`openclaw pairing approve <channel> <CODE>` 批准即可。

**Q：飞书配置好凭证后收不到消息？**  
请确认：① Gateway 已运行；② 飞书开放平台里事件订阅选的是「长连接」且已添加 `im.message.receive_v1`；③ 应用已发布、机器人在飞书里能搜到。若用 webhook 模式，需在配置里设 `verificationToken`，并在开放平台填写对应回调地址。详见 [Feishu 官方文档](https://docs.openclaw.ai/channels/feishu)。

**Q：飞书配对失败或消息路由不到助手（binding 不准）？**  
2026 年 2 月左右发布的版本中，可能出现**内置飞书插件**与**你手动安装的飞书插件**同时存在的情况，导致 binding 无法正确路由消息、配对难以成功。若你遇到「配置都对但消息/配对异常」，请检查插件目录（如 `~/.openclaw/plugins` 或安装路径下的 `plugins`）是否存在重复的飞书相关插件；**删除其中一份**（保留内置或保留自装的其一即可），重启 Gateway 后再试配对。实测删除多余的那一份后，配对即可正常完成。

**Q：Telegram 群聊里 @ 机器人没反应？**  
检查 BotFather 里是否开启了群组权限（如 `/setjoingroups` 允许加入群组）；群内是否设置了 `requireMention: true`（只有 @ 机器人才会回复）。若希望机器人看全群消息，需在 Telegram 里把机器人的「Privacy Mode」关掉（对 @BotFather 发 `/setprivacy`），并在群设置里给机器人相应权限。

**Q：WhatsApp 扫码后断线或收不到消息？**  
确保运行 Gateway 的电脑网络稳定；同一号码不要在多个地方同时登录 WhatsApp Web。若换设备或重装，需要重新执行 `openclaw channels login --channel whatsapp` 再扫一次码。

---

## 延伸阅读

- 飞书渠道完整说明（英文）：[Feishu](https://docs.openclaw.ai/channels/feishu)  
- Telegram 渠道（英文）：[Telegram](https://docs.openclaw.ai/channels/telegram)  
- WhatsApp 渠道（英文）：[WhatsApp](https://docs.openclaw.ai/channels/whatsapp)  
- 配对与访问控制（英文）：[Pairing](https://docs.openclaw.ai/channels/pairing)  
- 多语言文档（对非英文用户更友好）：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
