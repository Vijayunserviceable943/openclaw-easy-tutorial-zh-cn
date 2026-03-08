---
title: 给你的助手定规矩：一分钟搞懂配置文件
description: OpenClaw 配置文件 openclaw.json 位置与结构，改模型、工作区、环境变量与多环境；API Key 与模型白名单配置。
keywords: [OpenClaw, 配置文件, openclaw.json, 模型, 环境变量, API Key]
---

# 给你的助手定规矩：一分钟搞懂配置文件

读完这一节，你会知道 OpenClaw 的「规矩」都写在哪、配置文件**大致长什么样**，以及怎么改**模型**、**工作区**和常用选项；顺带了解几个环境变量，方便你在多环境或服务器上跑时心里有数。不要求你背下所有字段，会用、会查即可。

---

## 配置文件在哪、长什么样

OpenClaw 的配置写在**一个文件**里，默认路径是：

```
~/.openclaw/openclaw.json
```

（`~` 代表你的用户主目录。）文件格式是 **JSON5**：和普通 JSON 差不多，但允许注释、末尾多一个逗号等，写起来更宽松。若这个文件不存在，OpenClaw 会用内置的默认值，照样能跑；你只有想「定规矩」时才需要新建或改它。

**最简示例**（只改工作区 + 限制谁可以发 WhatsApp）：

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+86你的手机号"] } },
}
```

上面两行分别表示：助手的默认**工作区**目录；WhatsApp 只接受名单里号码发来的消息。其他没写的（例如用哪个模型、Gateway 端口）都用默认值。

---

## 配置文件里通常会有哪些块

你不需要一次全写，用到哪块就加哪块。常见几块可以这样理解：

| 块 | 作用 |
|----|------|
| **agents** | 和「助手本身」有关的默认值：工作区路径、用哪个模型、可选沙箱等。 |
| **channels** | 各**渠道**（飞书、Telegram、WhatsApp 等）的开关、白名单、配对策略等。 |
| **gateway** | **Gateway**（那座「桥」）的端口、认证方式、是否热加载配置等。 |
| **session** | 会话的隔离方式、重置策略等（进阶用）。 |

每一块下面还有子字段；完整列表在官方 [Configuration reference](https://docs.openclaw.ai/gateway/configuration-reference) 里。你改的时候可以只动你关心的那一小段，其余保持不动即可。

---

## 怎么改模型（provider/model）

助手背后用的**模型**（负责「想和说」的 AI）是在配置里定的，格式是：**provider/model**。例如：

- `anthropic/claude-sonnet-4-5`：Anthropic 家的 Claude Sonnet 4.5
- `openai/gpt-5.2`：OpenAI 的 GPT-5.2

**Provider**（提供商）就是提供这个模型的服务商；同一家可能有多个模型，所以用 `provider/model` 一起指定。API Key 等认证信息不写在这个文件里，而是放在环境变量或 OpenClaw 的**认证配置**里（向导 `openclaw onboard` 或 `openclaw configure` 会帮你填）。

在配置里设**主模型**和**备用模型**的示例：

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

- **primary**：默认用的主模型。
- **fallbacks**：主模型不可用时（例如超时、限流）依次尝试的模型。
- **models**：这里的键就是「允许用的模型列表」；写了之后，用户在对话里用 `/model` 切换时只能选这些。**alias** 是显示用的短名字（如 "Sonnet"）。若你不写 **models**，就不做限制；一写就变成**白名单**，不在名单里的模型会被拒绝，表现就是「选了某个模型却没反应」——这时要么把该模型加进 **models**，要么把 **models** 整块删掉恢复不限制。

模型名和提供商以官方文档和 [Model providers](https://docs.openclaw.ai/concepts/model-providers) 为准；不同 provider 的 API Key 用对应环境变量（如 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY`）或通过向导配置。

---

## 怎么改配置：四种方式

1. **直接改文件**  
   用任意文本编辑器打开 `~/.openclaw/openclaw.json`，改完保存。Gateway 会**监听这个文件**，多数改动会自动生效（热加载）；少数涉及服务本身的（例如端口）可能需要重启 Gateway：`openclaw gateway restart`。

2. **用命令行**  
   ```bash
   openclaw config get agents.defaults.workspace
   openclaw config set agents.defaults.heartbeat.every "2h"
   openclaw config unset tools.web.search.apiKey
   ```  
   适合快速改某一项或脚本里用。

3. **用 Control UI**  
   浏览器打开 `http://127.0.0.1:18789/`，在 **Config** 里用表单或「Raw JSON」改；保存后同样会热加载。

4. **用向导**  
   `openclaw onboard` 或 `openclaw configure` 会一步步问你并写进配置，适合第一次搭或大改。

注意：配置必须符合 OpenClaw 的 **schema**（结构规则）。若你手抖多写了未知字段或写错类型，Gateway 可能**拒绝启动**；此时只能跑诊断类命令（如 `openclaw doctor`、`openclaw logs`）。用 `openclaw doctor` 可以看到具体报错，按提示修配置文件或让 doctor 帮你修。

---

## 常用环境变量（多环境、服务器时有用）

若你在**服务器**上跑、或用**多套配置**（例如一个生产一个测试），会用到下面几个变量。不设的话，都用默认值。

| 变量 | 作用 |
|------|------|
| **OPENCLAW_CONFIG_PATH** | 指定配置文件路径，不用默认的 `~/.openclaw/openclaw.json`。 |
| **OPENCLAW_STATE_DIR** | 指定「状态目录」：会话、凭证、插件等都在这里，默认是 `~/.openclaw`。 |
| **OPENCLAW_HOME** | 内部解析路径时用的「主目录」，服务账号或容器里常用。 |
| **OPENCLAW_PROFILE** | 多环境时用：设成非 `default` 时，状态目录会变成 `~/.openclaw-<profile>`，工作区默认会变成 `~/.openclaw/workspace-<profile>`，便于同一台机器跑多套互不干扰。 |

例如跑两套 Gateway、用不同配置和状态目录：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json OPENCLAW_STATE_DIR=~/.openclaw-main openclaw gateway --port 18789
OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json OPENCLAW_STATE_DIR=~/.openclaw-rescue openclaw gateway --port 19001
```

多环境 / profile 的详细用法见官方 [Multiple gateways](https://docs.openclaw.ai/gateway/multiple-gateways) 或 [Environment](https://docs.openclaw.ai/help/environment)。

---

## 本节要点

- 配置文件默认是 **`~/.openclaw/openclaw.json`**，格式 **JSON5**；不写也能用默认值。
- 常见块：**agents**（工作区、模型等）、**channels**（飞书/Telegram/WhatsApp 等）、**gateway**（端口、认证）、**session**（会话策略）。
- 模型格式 **provider/model**；设 **agents.defaults.model.primary** 和 **fallbacks**；写 **agents.defaults.models** 会变成模型白名单。
- 改配置可：直接编辑文件（多数会热加载）、`openclaw config get/set`、Control UI、`openclaw configure`；配置不合法时 Gateway 可能不启动，用 **openclaw doctor** 查错。
- 多环境 / 服务器可用 **OPENCLAW_CONFIG_PATH**、**OPENCLAW_STATE_DIR**、**OPENCLAW_HOME**、**OPENCLAW_PROFILE**。

---

## 常见问题

**Q：改完配置没生效？**  
多数配置支持热加载，保存文件或 Control UI 保存即可。若改了端口、认证方式等，需要执行 `openclaw gateway restart`。若你用了 **OPENCLAW_CONFIG_PATH** 或 **OPENCLAW_PROFILE**，确认当前进程读的是你改的那份配置。

**Q：提示 "Model is not allowed" 或选了某模型没反应？**  
说明你配置了 **agents.defaults.models**（模型白名单），当前选的模型不在名单里。解决：把该模型加进 **models**，或删掉 **models** 取消白名单；可用 `/model list` 看当前允许的列表。

**Q：API Key 写在哪？**  
不要写在 `openclaw.json` 里。用环境变量（如 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY`）或通过 `openclaw onboard` / `openclaw configure` 写入 OpenClaw 的认证存储（在状态目录下），配置里只写模型名（provider/model）。

**Q：想在同一台机器跑两套互不影响的 OpenClaw？**  
用不同的 **OPENCLAW_CONFIG_PATH** 和 **OPENCLAW_STATE_DIR** 启动两个 Gateway，并指定不同端口（如 `--port 18789` 和 `--port 19001`）。详见 [Multiple gateways](https://docs.openclaw.ai/gateway/multiple-gateways)。

---

## 延伸阅读

- 配置总览与示例（英文）：[Configuration](https://docs.openclaw.ai/gateway/configuration)  
- 配置字段完整列表（英文）：[Configuration reference](https://docs.openclaw.ai/gateway/configuration-reference)  
- 模型选择与 provider（英文）：[Models](https://docs.openclaw.ai/concepts/models)、[Model providers](https://docs.openclaw.ai/concepts/model-providers)  
- 环境变量说明（英文）：[Environment](https://docs.openclaw.ai/help/environment)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
