---
course: Agentic AI
module: 5
lesson: 1
title: "Planning workflows"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/jcl177/planning-workflows
watched_at: 2026-07-14
reviewed_at:
tags: [planning, highly-autonomous-agents, tool-use, task-decomposition, runtime-control]
---

# Planning Workflows

## TL;DR

Planning design pattern 的核心是：不要让 developer 预先写死所有步骤，而是让 LLM 根据用户请求和可用工具，在 runtime 生成一个 step-by-step plan，然后按计划逐步执行。它适合用户请求变化大、工具组合顺序不固定的任务，例如 customer service、email assistant、coding agent。但它的代价是可控性下降：developer 运行前不知道模型会生成什么 plan，所以需要更强的 logging、guardrails 和 eval。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理；本次读取到 78 条去重 transcript cues。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- Module 5 关注更高 autonomy 的 agent pattern：系统可以自己决定下一步，而不是完全依赖 hard-coded workflow。
- Planning pattern 让 LLM 先输出 plan，再把 plan 的每一步交给 LLM/tool-use workflow 执行。
- Developer 提供的是工具集合、工具说明、上下文和执行框架；runtime 的步骤顺序由 LLM 规划。
- 一个 plan 通常不是最终答案，而是中间控制结构：它告诉 executor 下一步该查什么、调用什么、用什么结果继续。
- Planning 的价值在于处理更宽泛的请求分布；同一套 tools 可以被不同用户请求组合成不同 workflow。
- Planning 已经在高度 agentic 的 coding systems 中比较常见；在其他业务场景里仍然更偏探索阶段。

## Basic Workflow

课程里的 planning workflow 可以抽象成：

```text
user request
  -> LLM sees available tools and task context
  -> LLM writes a step-by-step plan
  -> executor runs step 1 with tools/context
  -> step 1 output becomes context for step 2
  -> executor runs step 2
  -> ...
  -> final LLM call synthesizes user-facing answer
```

这里的关键不是“模型想一想再回答”，而是把 plan 当成一个可执行的中间 artifact。每一步都可以触发工具调用，也可以把上一步的结果带进下一步。

## Example: Sunglasses Customer Service

课程用 sunglasses store 做例子。用户问：有没有圆形、库存中、低于 100 美元的太阳镜？

如果 developer hard-code，可能要预先写死一条固定流程。但真实客服问题会变化：有人问库存，有人问价格，有人要退货，有人要查历史订单。

Planning pattern 的做法是先给 LLM 一组工具，例如：

- get item descriptions
- check inventory
- get item price
- check past transactions
- process item sale
- process item return

然后让 LLM 为当前请求生成计划。对“圆形、库存中、低于 100 美元”这个请求，一个合理计划是：

1. 查商品描述，找出圆形太阳镜。
2. 查这些候选商品是否有库存。
3. 查有库存商品的价格，并筛出低于 100 美元的结果。
4. 汇总成给用户的回答。

如果用户换成“我要退掉之前买的金色镜框眼镜”，同一组工具可以被规划成另一条路径：先查历史购买，再查商品描述，识别金色镜框，再调用退货工具。

## Example: Email Assistant

第二个例子是 email assistant。用户说：回复 Bob 在 New York 的 dinner invitation，告诉他会参加，并把邮件归档。

可用工具可能包括：

- search email
- send email
- move email
- delete email

Planner 可以生成：

1. 搜索 Bob 发送、提到 dinner/New York 的邮件。
2. 生成并发送确认参加的回复。
3. 如果发送成功，把原邮件移动到 archive folder。

这个例子体现 planning 的两个点：

- 工具顺序不是固定的，而是由请求决定。
- 后一步依赖前一步结果，例如必须先找到目标邮件，才能回复和归档。

## Why Planning Is Powerful

Planning 适合这些情况：

- 用户请求变化大，无法枚举所有固定流程。
- 同一组 tools 需要按不同顺序组合。
- 任务需要跨多步积累中间结果。
- 复杂任务可以被拆成 checklist 式步骤逐个完成。
- 每一步都可以被记录、检查或失败重试。

它把 task decomposition 从 design time 推到 runtime：developer 不再提前拆好每一种流程，而是让 LLM 针对当前请求拆解。

## Control Tradeoff

Planning 的风险也很直接：系统更难控制。

在 deterministic workflow 里，developer 知道步骤顺序；在 planning workflow 里，developer 只知道可用工具和执行框架，不知道每次运行会生成什么 plan。

这意味着 planning workflow 特别需要：

- plan logging：保存每次生成的 plan。
- step-level trace：记录每一步输入、输出、工具调用和错误。
- guardrails：限制危险工具、敏感动作、写操作和高成本调用。
- plan eval：评估 plan 是否覆盖了必要步骤、顺序是否合理、是否包含多余或危险动作。
- end-to-end eval：确认最终答案或动作真的满足用户目标。

## Data Analyst Translation

如果做数据分析 agent，planning 可以帮助处理开放式分析问题，例如：

> 帮我分析最近三个月新用户留存下降的原因，并给出可能的产品动作。

一个 planning agent 可能会自己规划：

1. 明确指标口径：新用户、留存窗口、时间范围。
2. 查询总体趋势，确认下降发生在哪些时间段。
3. 按渠道、平台、地区、版本拆分。
4. 检查产品发布、活动、数据口径变更等背景事件。
5. 对异常分组生成假设。
6. 输出证据、局限和下一步实验建议。

这比固定的 “SQL -> chart -> summary” 更灵活，但也更危险：它可能跳过关键 sanity check，可能选错拆分维度，也可能调用过多昂贵查询。所以数据分析 planning agent 必须记录 plan、SQL、结果、解释和每步 rationale，并用 eval 检查是否遵守分析 SOP。

## Design Takeaways

- Planning 是提高 autonomy 的重要 pattern，但不是默认选择。
- 如果 workflow 步骤稳定、风险高、合规要求强，hard-coded workflow 往往更适合。
- 如果请求空间很宽、工具组合多、步骤顺序依赖上下文，planning 更有价值。
- Planning 和 tool use 是互补的：tools 提供能力，planning 决定如何编排这些能力。
- Plan 应该被当作可观察、可评估的中间产物，而不是隐藏在模型内部。
- Coding agent 里的 checklist/build-plan 是 planning pattern 的典型成功场景。
- 越高 autonomy，越需要 eval、trace、权限控制和 human approval。

## Failure Modes

- Plan 漏掉必要步骤，例如没有先查库存就直接回答。
- Plan 顺序错误，例如先发邮件再确认目标邮件。
- Plan 过度复杂，调用不必要工具，增加 latency 和 cost。
- Plan 使用了不该用的工具，例如高风险写操作或删除操作。
- Executor 没有正确继承前一步输出，导致后续步骤基于错误上下文执行。
- Plan 看起来合理，但最终答案没有满足用户真实意图。
- Developer 没有记录 plan 和 step traces，出错后无法 debug。
- Planning 被用于本来应该 hard-code 的稳定高风险流程。

## Implementation Hooks

- Planner prompt: 明确可用 tools、用户目标、约束、禁止动作和输出 plan schema。
- Plan schema: 使用结构化格式，例如 `steps[]`，每步包含 `goal`、`inputs_needed`、`allowed_tools`、`success_criteria`。
- Executor loop: 每次只执行当前 step，把 step result 写回上下文，再进入下一步。
- Step trace: 记录 step text、tool calls、tool arguments、tool results、latency、cost、errors。
- Guardrail layer: 对发送邮件、退款、删除、数据库写入等动作加 human approval。
- Plan evaluator: 检查 plan 是否完整、顺序是否合理、是否包含禁用动作。
- Fallback: 如果 plan 不确定或触发高风险工具，降级到 human review 或 deterministic workflow。

## My Notes

- 这节把 Module 1 的 autonomy spectrum、Module 3 的 tool use、Module 4 的 eval/trace 串起来了。
- 对我来说，planning 不是“更聪明的 prompt”，而是一个新的 orchestration pattern：先生成 workflow，再执行 workflow。
- Planning 最适合我未来的数据分析 agent 里的开放式诊断问题；但生产化时要把 plan、SQL 和每步 evidence 全部留痕。
- 这节也解释了为什么 coding agent 会先列 checklist 再逐步改代码：那其实就是 planning pattern 的落地版本。

## Open Questions

- Plan quality 应该怎么评估：完整性、顺序、工具选择、风险控制，还是最终结果？
- 对数据分析 agent，哪些步骤必须固定，哪些步骤可以交给 planner 动态决定？
- 如果 planner 生成了错误 plan，应该让 reflection 修 plan，还是直接 fallback 到 human review？
- Planner 和 executor 应该是同一个模型，还是分别用不同模型？
- 如何限制 planning agent 不要为了“看起来全面”而过度调用工具？

## Related

- [Planning pattern](../../concepts/planning-pattern.md)
- [Agentic design patterns](../../concepts/agentic-design-patterns.md)
- [Autonomy spectrum](../../concepts/autonomy-spectrum.md)
- [Task decomposition](../../concepts/task-decomposition.md)
- [Tool use](../../concepts/tool-use.md)
- [Agentic evals](../../concepts/agentic-evals.md)

## Next Actions

- [x] 新增 Module 5 README / summary。
- [x] 新增 planning pattern 概念卡。
- [x] 更新 glossary / open questions / reference map。
- [ ] 下一节继续看 `Creating and executing LLM plans`，把 plan/executor 的具体实现机制补上。
