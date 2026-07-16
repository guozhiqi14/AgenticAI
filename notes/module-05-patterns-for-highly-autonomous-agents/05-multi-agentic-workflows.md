---
course: Agentic AI
module: 5
lesson: 5
title: "Multi-agentic workflows"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/608l3n/multi-agentic-workflows
watched_at: 2026-07-16
reviewed_at:
tags: [multi-agent, collaboration, role-decomposition, manager-agent, communication-patterns]
---

# Multi-Agentic Workflows

## TL;DR

Multi-agentic workflow 把复杂任务拆给多个带不同 role prompt、工具权限和职责边界的 agents 协作完成。它的价值不是“多叫几次同一个 LLM”，而是让 developer 可以像设计一个团队一样拆解任务：researcher 做调研，graphic designer 做视觉，writer 做文案，manager agent 负责任务分派和协调。多 agent 能提升 modularity、复用性和协作开发效率，但代价是 communication pattern、debug、eval 和协调成本都会上升。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理；本次读取到 129 条去重 transcript cues。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- Multi-agent workflow 用多个 agents 协作完成复杂任务，而不是让一个 agent 包办所有步骤。
- 多 agent 的核心设计思路类似人类团队：不同角色负责不同子任务。
- 每个 agent 可以有不同 role prompt、tools、输入输出格式和 eval 标准。
- 常见拆分方式来自现实工作分工，例如 researcher、graphic designer、writer、editor、statistician、investigator。
- Multi-agent workflow 可以是简单线性链路，也可以有 manager agent 协调多个 specialist agents。
- 下一节会进一步展开关键设计问题：不同 agents 之间如何通信。

## Why Multiple Agents?

第一反应可能是：如果底层都是 LLM，为什么不只用一个 agent？

课程给了两个类比：

- 计算机里即使只有一个 CPU，developer 也会把工作拆成多个 process/thread，方便组织程序。
- 一个复杂业务任务通常不是招一个全能人，而是组一个团队，让不同人承担不同角色。

因此 multi-agent 的价值不只是“模型更多”，而是让任务拆解更自然：

- 每个 agent 的 prompt 更聚焦。
- 每个 agent 只需要一部分工具。
- 每个 agent 可以单独调试和优化。
- 不同开发者可以并行构建不同 agent。
- 成熟 agent 有机会跨应用复用。

## Example: Marketing Assets Team

课程用 sunglasses marketing brochure 做例子。目标是为太阳镜做营销素材。

一个单 agent 也许能直接写 brochure，但多 agent 可以按团队角色拆：

| Agent | Responsibility | Likely Tools |
| --- | --- | --- |
| Researcher | 分析市场趋势、竞品和用户偏好 | web search, web fetch, internal docs |
| Graphic Designer | 生成图表、视觉素材、产品图片变体 | image generation/manipulation, code execution for charts |
| Writer | 把 research 和 visual assets 组织成文案 | LLM text generation |

这里每个 agent 的能力边界都不一样：researcher 需要搜索工具，designer 需要图像/图表工具，writer 可能不需要额外工具，只需要高质量写作 prompt。

## Example: Research Article Team

如果任务是写 research article，可以拆成：

- Researcher: 搜索资料、整理证据。
- Statistician: 做统计计算和数据检验。
- Lead writer: 写初稿和结构。
- Editor: 润色、统一风格、检查逻辑。

这和 Module 4 的 eval/error analysis 也能连起来：不同 agent 的输出可以单独观察、单独评价，而不是只看最终文章。

## Linear Multi-Agent Workflow

一种最简单的 communication pattern 是线性链路：

```text
user request
  -> researcher
  -> graphic designer
  -> writer
  -> final output
```

以 sunglasses marketing campaign 为例：

1. Researcher 输出趋势和竞品报告。
2. Graphic designer 根据 research 生成数据可视化和视觉素材。
3. Writer 使用 research + visuals 写最终 brochure。

优点：

- 容易理解和实现。
- 每一步输入输出明确。
- Debug 相对简单。
- 每个 agent 可以独立优化。

缺点：

- 下游强依赖上游质量。
- 如果早期 agent 走偏，后面可能一路放大错误。
- 不适合需要多轮协商或并行探索的任务。

## Manager-Agent Workflow

另一种模式是让一个 manager agent 负责任务规划和委派：

```text
user request
  -> manager agent
      -> researcher agent
      -> graphic designer agent
      -> writer agent
  -> manager reviews / reflects
  -> final output
```

这里 manager agent 很像前面 planning pattern 里的 planner，只是它可调用的不是普通 tools，而是一组 specialist agents。

换句话说：

```text
planning with tools:
  LLM chooses tools

planning with agents:
  manager LLM chooses specialist agents
```

这个视角很重要：agent 可以被当作更高层级的 tool。区别是 agent 内部也可能有自己的 prompt、tools、memory、eval 和 workflow。

## Design Takeaways

- Multi-agent 的出发点应该是 role decomposition，而不是为了显得复杂。
- 每个 agent 应该有清楚的 responsibility、tools、input/output contract 和 eval。
- 先从线性 workflow 开始通常更容易 debug。
- Manager agent 适合任务需要动态委派、整合和 review 的场景。
- Specialist agent 可以跨应用复用，但前提是接口和职责足够稳定。
- Multi-agent 增加 modularity，也增加 communication、latency、cost 和 observability 负担。
- 设计 multi-agent 系统时，communication pattern 是核心架构决策。

## Failure Modes

- Role boundary 模糊，多个 agent 重复做同一件事或互相等待。
- 上游 agent 输出质量差，下游 agent 无法恢复。
- Manager agent 委派错误，把任务交给不合适的 specialist。
- 每个 agent 都能调用过多工具，导致权限和行为不可控。
- Agent 之间传递上下文过长，成本和噪音上升。
- 没有 per-agent trace，最终出错时不知道是哪一环坏了。
- 多 agent coordination 比任务本身还复杂，得不偿失。

## Implementation Hooks

- Agent registry: 维护 `agent_name -> role prompt -> tools -> input/output schema`。
- Role prompt: 每个 agent 明确身份、职责、禁止事项和交付物格式。
- Communication contract: 明确上游输出如何传给下游。
- Manager prompt: 描述可用 agents、每个 agent 的能力和何时调用。
- Trace: 记录每个 agent 的 input、output、tool calls、latency、cost 和错误。
- Eval: 同时做 final output eval 和 per-agent/component eval。
- Guardrails: 对高风险 agent 或 tool call 加审批。
- Reuse: 只有当 agent 的接口稳定时，再把它提升为跨 workflow 的 reusable agent。

## Data Analyst Translation

如果做数据分析 agent，可以考虑这些角色：

| Agent | Responsibility |
| --- | --- |
| Metric analyst | 查指标口径、确认业务定义 |
| SQL analyst | 写 SQL、执行查询、检查结果 |
| Statistician | 做统计检验、异常检测、置信区间 |
| Visualization designer | 选择图表类型并生成图 |
| Insight writer | 写结论、限制条件和行动建议 |
| Manager agent | 拆任务、调度 specialist、做最终 review |

但第一版不一定要多 agent。更稳妥的路线是：

1. 先做一个 fixed workflow 或 single-agent planning prototype。
2. 用 traces 找到哪个步骤最复杂、最常错。
3. 只有当某个步骤足够独立且值得复用时，再拆成 specialist agent。

## My Notes

- 这节把 task decomposition 推到“组织设计”的层面：不是只拆步骤，而是拆角色。
- 我不应该把 multi-agent 当默认方案；它适合职责边界清楚、工具权限不同、或者团队式协作自然存在的任务。
- 对数据分析 agent，manager + specialist 可能有价值，但第一版最好还是保持简单，先把 trace/eval 做扎实。
- 下一节 communication patterns 会很关键，因为多 agent 真正难点不是创建多个 prompt，而是让它们如何交换信息和承担责任。

## Open Questions

- 什么情况下应该从 single-agent planning 升级到 multi-agent workflow？
- Specialist agent 的接口应该如何设计，才能被不同 workflow 复用？
- Manager agent 的委派质量应该怎么 eval？
- Multi-agent trace 应该如何展示，才能快速定位是哪一个 agent 出错？
- 对数据分析 agent，哪些角色值得真的拆成 agent，哪些只是 workflow step？

## Related

- [Planning workflows](01-planning-workflows.md)
- [Communication patterns for multi-agent systems](07-communication-patterns-for-multi-agent-systems.md)
- [Planning pattern](../../concepts/planning-pattern.md)
- [Agentic design patterns](../../concepts/agentic-design-patterns.md)
- [Task decomposition](../../concepts/task-decomposition.md)
- [Tool use](../../concepts/tool-use.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Multi-agent collaboration](../../concepts/multi-agent-collaboration.md)
- [Multi-agent communication patterns](../../concepts/multi-agent-communication-patterns.md)

## Next Actions

- [x] 更新 Module 5 README / summary。
- [x] 新增 multi-agent collaboration 概念卡。
- [x] 增加 glossary / open questions。
- [x] 已继续看 `Communication patterns for multi-agent systems`，整理 linear、hierarchical、deeper hierarchy、all-to-all 等通信模式。
