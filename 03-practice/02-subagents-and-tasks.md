---
title: 派助手去「跑腿」：子 Agent 与并行任务
description: OpenClaw 子 Agent（Sub-Agent）概念、/subagents spawn、sessions_spawn 工具、allowAgents、announce 与并行任务。
keywords: [OpenClaw, 子 Agent, Sub-Agent, sessions_spawn, subagents spawn, 并行任务]
---

# 派助手去「跑腿」：子 Agent 与并行任务

读完这一节，你会搞懂**子 Agent**（Sub-Agent）是什么、什么时候用，以及怎样用 **`/subagents spawn`** 或工具 **sessions_spawn** 派一个「后台小助手」去干活，等它跑完再把结果**报回**当前对话；还能用不同模型或不同 Agent 做**并行研究、分工**，主助手不卡住。

---

## 子 Agent 是什么、为啥要用

**子 Agent**（Sub-Agent）是一次**在后台跑的助手会话**：由你当前对话里的主助手（或你本人）发起，在单独的一个会话里执行你给的任务，**跑完后**把结果「汇报」回你当前这条对话里。

可以把它想成：主助手在跟你聊天，你或主助手说「去查一下某件事、算一下某笔账」，主助手就**派一个小分身**去专门做那件事；小分身在自己的会话里干活，不占着你当前对话；干完了会把结论或摘要发回来，主助手（或你）再根据结果继续聊。

适合用的场景举例：

- **耗时的调研、汇总**：让子 Agent 去搜资料、读长文、做摘要，主对话不卡住。
- **并行多件事**：同时派两三个子 Agent 各做一件事，最后一起看结果。
- **不同「角色」或模型**：主助手用强模型，子任务用便宜一点的模型；或派到另一个 Agent（例如专门配了「法务」人设的 Agent）去做专业判断，再报回主会话。

子 Agent 有**自己的会话**（存在 `agent:<agentId>:subagent:<uuid>` 这种 key 下），和主会话隔离；默认不会拿到「查别的会话、派新的子 Agent」这类会话工具，避免乱套。跑完会通过 **announce**（汇报）把结果发回发起方。

---

## 怎么派一个子 Agent 去跑腿（用户侧）

在**支持斜杠命令的渠道**（如 Control UI、部分 IM）里，你可以直接发：

```
/subagents spawn main 帮我把这段需求整理成三条用户故事，每条一两句话
```

或指定用哪个 Agent、换模型：

```
/subagents spawn coding 用 Python 写一个读 CSV 并求列平均的函数 --model anthropic/claude-3-5-haiku
```

- **spawn** 后面第一个参数是 **agentId**（用哪个「脑子」跑这个任务），例如 `main`、`coding`。
- 后面是**任务描述**（一句话或一段话）。
- **--model**、**--thinking** 可选，只对这一趟子 Agent 生效；不写就沿用该 Agent 的默认模型。

这条命令是**非阻塞**的：发出去后立刻返回一个 **run id**，子 Agent 在后台跑；跑完后会有一条**汇报消息**发回当前对话（结果摘要、状态、简单统计）。你可以在等待时继续跟主助手聊别的事；想看子 Agent 的详细输出，可以用：

- `/subagents list` — 看当前会话派出的子 Agent 列表与状态  
- `/subagents log <id>` — 看某次的日志  
- `/subagents info <id>` — 看某次的元信息（状态、会话 key、 transcript 路径等）  
- `/subagents kill <id>` — 提前结束某次子 Agent

---

## 主助手在对话里怎么派（sessions_spawn）

主助手在回复过程中也可以**自己**派子 Agent，用的是工具 **sessions_spawn**（由模型在推理时调用）。参数里会带：

- **task**：要子 Agent 执行的任务描述（必填）。
- **agentId**：用哪个 Agent 跑；不写一般就是当前 Agent。
- **model**、**thinking**、**runTimeoutSeconds**：可选，覆盖这一趟的模型、思考深度、超时。

派出去后，子 Agent 在后台跑；跑完会通过 **announce** 把结果注入回主助手的上下文，主助手再根据结果继续回复你。所以你可以对主助手说「你派个人去查一下 X，查完再总结给我」，主助手就会在合适的时机调 sessions_spawn，等子 Agent 回报后再总结。

**谁可以被派**：默认只允许派到**当前同一个 Agent**；若你配了多个 Agent（如 main、coding、legal），可以在配置里用 **agents.list[].subagents.allowAgents** 放开（例如 `["*"]` 表示允许派到任意已配置的 Agent），这样主助手才能把任务派给「法务 Agent」等。

---

## 给子 Agent 用不同模型、控制成本

每个子 Agent 都是一次**完整对话**，会单独消耗 Token。若你经常派子 Agent 做杂活，可以：

- 在配置里设 **agents.defaults.subagents.model**（或某个 Agent 下的 **subagents.model**），让子 Agent 默认用更便宜或更快的模型；
- 主助手继续用强模型，只在「派出去跑腿」时用便宜模型，省成本。

在 `/subagents spawn` 或 sessions_spawn 里传 **model** 可以单次覆盖，适合「这一趟用 Haiku」「那一趟用 Sonnet」的灵活用法。

---

## 本节要点

- **子 Agent** = 一次在后台跑的助手会话，跑完后通过 **announce** 把结果报回发起方；适合耗时任务、并行多任务、或让「另一个 Agent/另一个模型」专门干一件事。
- **用户派**：在支持斜杠命令的地方发 **/subagents spawn 某个agentId 任务描述**，可加 **--model**、**--thinking**；非阻塞，跑完会有汇报消息。
- **主助手派**：通过工具 **sessions_spawn**（task、agentId、model 等）；谁可被派由 **agents.list[].subagents.allowAgents** 控制。
- **查看与控制**：/subagents list、log、info、kill；子 Agent 有独立会话，默认不带会话类工具。
- **成本**：可为子 Agent 设默认 **subagents.model**，或用 spawn 时传 model，让跑腿用便宜模型。

---

## 常见问题

**Q：派出去后没看到汇报？**  
汇报是**尽力而为**的：若 Gateway 中途重启或异常，未完成的 announce 可能丢失。正常情况下跑完会有一条结果消息；可用 `/subagents list` 看状态，用 `/subagents log <id>` 看该次输出。

**Q：能派到「法律」「会计」那种别的 Agent 吗？**  
可以，但要先在 **agents.list** 里配好那个 Agent（见 3.1），再在该主助手的配置里设 **subagents.allowAgents**，把那个 Agent 的 id 加进去（或 `["*"]` 允许所有）；spawn 时写那个 agentId 即可。

**Q：子 Agent 默认能用哪些工具？**  
默认拥有**除会话类**（sessions_list、sessions_history、sessions_send、sessions_spawn）以外的工具；具体可通过 **tools.subagents.tools.allow/deny** 再细调。详见官方 [Sub-Agents](https://docs.openclaw.ai/tools/subagents)。

**Q：可以子 Agent 再派子 Agent 吗？**  
可以，但需在配置里把 **agents.defaults.subagents.maxSpawnDepth** 设为 2 或更大（默认 1）；深度 2 时适合「主助手 → 协调子 Agent → 多个干活子 Agent」的模式。详见官方文档 Nested Sub-Agents。

---

## 延伸阅读

- 子 Agent 完整说明（英文）：[Sub-Agents](https://docs.openclaw.ai/tools/subagents)  
- 斜杠命令（英文）：[Slash commands](https://docs.openclaw.ai/tools/slash-commands)  
- 多语言文档：[www.openclawx.cloud](https://www.openclawx.cloud)

[← 返回目录](../README.md)
