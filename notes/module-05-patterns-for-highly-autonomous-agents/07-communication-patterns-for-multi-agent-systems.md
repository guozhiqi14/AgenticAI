---
course: Agentic AI
module: 5
lesson: 7
title: "Communication patterns for multi-agent systems"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/gymk4l/communication-patterns-for-multi-agent-systems
watched_at: 2026-07-16
reviewed_at:
tags: [multi-agent, communication-patterns, linear-handoff, hierarchy, all-to-all]
---

# Communication Patterns for Multi-Agent Systems

## TL;DR

Multi-agent system 难点不只是“有几个 agent”，而是 agents 之间怎么通信、谁把结果交给谁、谁负责最终整合。课程介绍了四种常见/可见的 communication patterns：linear、hierarchical、deeper hierarchy 和 all-to-all。越往后越灵活，也越难预测、难 debug、难 eval。实际落地时应该优先从 linear 或单层 manager hierarchy 开始，把 trace、handoff contract 和 stopping condition 做清楚，再考虑更复杂的模式。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理；本次读取到 48 条去重 transcript cues。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- Multi-agent communication pattern 类似组织结构图：角色不够，通信路径也要设计。
- 最常见的两类是 linear handoff 和 hierarchical/manager coordination。
- Linear pattern 让 agent 按固定顺序传递结果，最简单、最容易 debug。
- Hierarchical pattern 让 manager agent 协调多个 specialist agents，适合需要动态委派和整合的任务。
- Deeper hierarchy 允许 specialist agent 再调用 sub-agents，但复杂度明显上升。
- All-to-all 允许任意 agent 和任意 agent 通信，探索性强，但输出更难预测。
- 选择 communication pattern 本质是在 control、flexibility、debuggability、latency/cost 之间取舍。

## Pattern 1: Linear Handoff

线性模式像流水线：

```text
user request
  -> researcher
  -> graphic designer
  -> writer
  -> final output
```

在 marketing brochure 例子里：

1. Researcher 做市场/竞品/用户偏好分析。
2. Graphic designer 根据 research 生成视觉资产。
3. Writer 接收 research 和 visual assets，写最终文案。

优点：

- 实现简单。
- 每一步输入输出清楚。
- Trace 容易看。
- 出错时比较容易定位是哪一环坏了。

缺点：

- 下游强依赖上游质量。
- 中途如果需要返工或协商，线性链路会变笨重。
- 不适合多个 agent 需要并行讨论的任务。

## Pattern 2: Hierarchical / Manager Coordination

单层 hierarchy 让 manager agent 做调度：

```text
user request
  -> manager
      -> researcher
      -> graphic designer
      -> writer
  -> manager integrates / reviews
  -> final output
```

这里 manager 决定什么时候调用 researcher、什么时候把结果交给 designer、什么时候让 writer 收尾。课程里强调，实际实现时通常更简单的做法是 specialist 把结果传回 manager，再由 manager 决定下一步，而不是让 researcher 直接把结果同时发给 designer 和 writer。

这个模式适合：

- 任务顺序不完全固定。
- Manager 需要根据中间结果决定下一步。
- 最终输出需要一个中心角色整合和 review。
- Specialist agents 的职责边界比较清楚。

风险：

- Manager 委派错了，整个 workflow 会走偏。
- Manager 成为 bottleneck。
- 所有上下文都回流到 manager，容易变长、变贵。
- 如果 trace 不清楚，很难判断是 specialist 错，还是 manager 整合错。

## Pattern 3: Deeper Hierarchy

Deeper hierarchy 是“组织结构图继续往下长”：

```text
manager
  -> researcher
      -> web researcher
      -> fact checker
  -> graphic designer
  -> writer
      -> style writer
      -> citation checker
```

它的价值是可以继续拆更细的专业角色，例如 researcher 自己再找 web researcher 和 fact checker 帮忙。

但这比单层 manager hierarchy 难很多：

- 调用链更长。
- Trace 更复杂。
- Context 传递更容易膨胀。
- Latency/cost 更难控制。
- 出错定位从“哪一步错了”变成“哪一层、哪个 agent、哪次 handoff 错了”。

所以它更适合已经有稳定单层 hierarchy、并且某个 specialist 内部确实复杂到值得拆成 sub-agents 的场景。

## Pattern 4: All-to-All Communication

All-to-all 允许任意 agent 和任意 agent 通信：

```text
researcher <-> designer
researcher <-> writer
designer   <-> writer
manager    <-> everyone
```

这很像一个开放讨论群。每个 agent 都知道还有哪些 agents 可以联系；当某个 agent 发消息时，消息进入接收方上下文，接收方再决定什么时候回应。最终可以用“所有 agent 都声明完成”或“writer/manager 判断足够好”作为停止条件。

优点：

- 灵活。
- 适合探索、头脑风暴、多视角协作。
- 有机会产生单一路径想不到的想法。

缺点：

- 很难预测。
- 很难 debug。
- 容易循环、重复、跑题。
- stopping condition 设计困难。
- 对高控制要求的业务场景不友好。

课程的实际建议偏谨慎：all-to-all 可以用于能容忍混乱和不确定性的探索任务，例如营销创意草稿；但如果任务要求可控、可复现、可解释，就要非常小心。

## Design Takeaways

- Communication pattern 是 multi-agent 架构里的核心设计决策。
- 默认从 linear 或单层 manager hierarchy 开始，不要一上来 all-to-all。
- 每条 handoff 都应该有明确 contract：传什么、格式是什么、谁负责 review。
- Manager hierarchy 更灵活，但要 eval manager 的 delegation 和 integration quality。
- Deeper hierarchy 只有在 specialist 内部复杂度足够高时才值得引入。
- All-to-all 需要更强 stopping condition、message trace 和 chaos budget。
- 如果最终输出错了，trace 要能回答：是哪一个 agent、哪一次通信、哪一个中间 artifact 导致的？

## Failure Modes

- Linear workflow 早期输出错误，后续 agent 继续放大错误。
- Manager agent 把任务派给错误 specialist，或者过早进入最终整合。
- Deeper hierarchy 里 sub-agent 输出无法被上层正确吸收。
- All-to-all 里 agents 互相聊天但不收敛。
- Handoff contract 不清楚，接收方误解上游输出。
- 每次通信都携带过多上下文，导致成本、噪音和 latency 上升。
- 没有 message-level trace，只能看到 final output，看不到协作过程。

## Implementation Hooks

- Handoff schema: 为每种 agent-to-agent message 定义结构化字段。
- Routing policy: 决定谁可以发消息给谁，哪些路径被禁止。
- Manager delegation log: 记录 manager 为什么调用某个 specialist。
- Message trace: 记录 sender、receiver、message type、artifact、timestamp、cost。
- Stop condition: 定义何时结束协作，例如 manager approve、writer finalize、max rounds、all agents done。
- Eval: 分开评估 delegation、handoff quality、specialist output、final integration。
- Guardrails: 对 all-to-all 或 deeper hierarchy 设置 max depth、max messages、max cost。

## Data Analyst Translation

数据分析 agent 也可以映射这几种模式：

| Pattern | Data Analyst Example | 建议 |
| --- | --- | --- |
| Linear | metric definition -> SQL query -> chart -> insight writing | 第一版最稳 |
| Manager hierarchy | manager 分派给 SQL analyst、statistician、visualization designer、writer | 适合复杂分析任务 |
| Deeper hierarchy | statistician 再调用 anomaly detector / power analysis checker | 只有稳定后再拆 |
| All-to-all | 多个 analyst agents 互相挑战假设和结论 | 适合探索，不适合自动发报告 |

对数据分析场景，最实用的路线还是：

1. 先用 fixed workflow 或 linear handoff。
2. 记录每一步 artifact 和错误。
3. 当某一步需要动态判断时，再引入 manager。
4. 只有当某个 specialist 内部真的复杂，才继续拆 sub-agents。
5. All-to-all 只放在 brainstorming / hypothesis generation，不直接进入生产结论。

## My Notes

- 这节把 multi-agent 从“角色拆分”推进到“组织结构设计”：谁和谁说话，比有几个 agent 更关键。
- 我做数据分析 agent 时，不应该默认做一个群聊式多 agent；更现实的是先做 linear pipeline，再把最需要动态调度的部分交给 manager。
- All-to-all 很像多人自由讨论，适合创意探索；但数据分析报告、SQL 生成、业务指标解释这类任务还是需要更强控制。
- Communication pattern 的选择应该和 eval/trace 一起设计，否则复杂度会超过收益。

## Open Questions

- 如何判断一个任务应该用 linear，而不是 manager hierarchy？
- Manager agent 的 delegation quality 应该如何单独 eval？
- All-to-all 的 stopping condition 怎样设计才不会无限讨论或过早收敛？
- Multi-agent message trace 应该怎么可视化，才能帮助 debug？
- 数据分析 agent 里，哪些 handoff 应该强制结构化，哪些可以保留自然语言？

## Related

- [Multi-agentic workflows](05-multi-agentic-workflows.md)
- [Planning workflows](01-planning-workflows.md)
- [Multi-agent collaboration](../../concepts/multi-agent-collaboration.md)
- [Multi-agent communication patterns](../../concepts/multi-agent-communication-patterns.md)
- [Task decomposition](../../concepts/task-decomposition.md)
- [Agentic evals](../../concepts/agentic-evals.md)

## Next Actions

- [x] 更新 Module 5 README / summary。
- [x] 新增 multi-agent communication patterns 概念卡。
- [x] 增加 glossary / open questions。
- [ ] 后续如果补 `Market Research Team Code Example`，重点对照真实 notebook 里采用了哪种 communication pattern。
- [ ] 进入 Module 5 quiz / graded assignment 前，复习 planning、code-as-plan、multi-agent communication 三条主线。
