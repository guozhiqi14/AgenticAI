---
course: Agentic AI
module: 4
lesson: 8
title: "Development process summary"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/hl2aj7/development-process-summary
watched_at: 2026-07-14
reviewed_at:
tags: [development-process, evals, error-analysis, traces, custom-evals, iteration]
---

# Development Process Summary

## TL;DR

构建 agentic workflow 不是线性流程，而是在 build 和 analyze 之间来回切换：先快速做一个 end-to-end prototype，读 outputs/traces，凭直觉和证据找到薄弱点；随着系统成熟，加入小 eval set、error analysis、component-level evals 和 cost/latency monitoring。经验不足的团队常常只忙着 build，不花足够时间 analyze，结果改进方向容易发散。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- Agentic AI 开发有两类同等重要的活动：build 和 analyze。
- Build 是写代码、调 prompt、换模型、改工具、优化 workflow。
- Analyze 是读 outputs/traces、做 error analysis、建 evals、判断下一步该改哪里。
- 初期可以先做 quick and dirty end-to-end system，用真实输出暴露 failure modes。
- 系统成熟后，再逐步加入 10 到 20 个样例的小 eval set、error analysis、component-level evals。
- 这个过程不是线性的：可能先调 end-to-end，再做 error analysis，再优化 component，再回到整体 eval。
- 通用 monitoring/logging 工具有帮助，但 agentic workflow 通常很 custom，所以仍然需要为具体应用构建 custom evals。

## Development Loop

课程把成熟的开发过程描述成一个来回循环：

```text
quick end-to-end prototype
  -> inspect outputs and traces
  -> identify weak components
  -> build small eval set
  -> run end-to-end metrics
  -> do error analysis
  -> build component-level evals
  -> improve selected components
  -> re-run end-to-end eval
```

关键点：analysis 不是“没在写代码所以没进展”。它是在决定下一小时、下一天、下一周应该把 build effort 放在哪里。

## Maturity Stages

| Stage | What You Do | Main Signal |
| --- | --- | --- |
| Quick prototype | 快速搭 end-to-end workflow | final outputs and traces |
| Early iteration | 手动读少量 outputs/traces | gut sense of weak components |
| First eval set | 10-20 examples + simple metrics | end-to-end quality trend |
| Error analysis | 给失败样例做 component attribution | which component fails most often |
| Component optimization | 为关键 component 建局部 eval | prompt/model/tool/parameter comparison |
| Production tuning | 记录 runtime、cost、latency | quality + cost + latency dashboard |

这和前面几节正好连起来：evals 给整体信号，error analysis 找原因，component-level eval 加速局部优化，latency/cost measurement 帮助生产化。

## Build vs Analyze

Build examples:

- 写 workflow code。
- 改 prompt 和 examples。
- 换模型、工具或 provider。
- 调 RAG/search/code execution 参数。
- 优化 latency/cost。

Analyze examples:

- 读 final outputs。
- 读 traces/spans。
- 统计 recurring failure modes。
- 建 end-to-end eval。
- 做 component attribution。
- 设计 custom evals。

经验不足的团队容易把时间几乎都花在 build 上。但如果没有 analysis，build 只是随机试错；有了 analysis，build 才会集中到最值得修的地方。

## Custom Evals

课程提醒：市面上有很多 trace monitoring、logging runtime、cost tracking 的工具，它们有价值，但多数 agentic workflows 都很 custom。

因此，真正关键的 eval 往往也需要 custom：

- 针对你的业务 failure mode。
- 针对你的 workflow components。
- 针对你的用户关心的质量标准。
- 针对你的 trace/span 结构。

工具可以帮你记录和展示，但“什么叫好、什么叫错、错在哪里、优先修什么”通常要自己定义。

## Data Analyst Translation

如果我做数据分析 agent，development loop 可以这样落地：

1. 快速做一个能从问题到 SQL 到图表/解释的 end-to-end prototype。
2. 手动看 10 到 20 个真实业务问题的输出。
3. 读 trace：schema retrieval、SQL generation、SQL execution、chart spec、final explanation 各自怎么表现。
4. 给失败样例打标签：schema 错、SQL 口径错、数据质量错、图表错、解释过度等。
5. 先做一个 end-to-end eval set，再为最常错的 component 建 component-level eval。
6. 优化最值得修的 component。
7. 再回到 end-to-end eval，确认整体分析质量真的变好。

这比“继续调总 prompt”更像工程。

## Design Takeaways

- 快速 end-to-end prototype 的价值不是完美，而是尽快拿到真实 outputs/traces。
- Analysis 是 build 的导航系统；没有 analysis，优化方向很容易漂。
- 小 eval set 可以先启动迭代，不必等到有大型 benchmark。
- Error analysis 让你知道哪个 component 最值得修。
- Custom evals 是 agentic workflow 成熟度的重要标志。
- 工具能帮你监控 traces/cost/runtime，但不能替你定义业务质量。

## Failure Modes

- 一直 build，不停改代码/prompt，但没有系统读 traces 和失败样例。
- 过早追求完美 eval framework，迟迟没有 end-to-end prototype。
- 只看 final outputs，不看中间 component，导致不知道该改哪里。
- 只依赖现成 monitoring tool，没有为自己的业务 failure modes 建 custom evals。
- 建了 component-level eval 后，不再回到 end-to-end eval。
- 把 development process 当线性步骤，而不是持续循环。

## Implementation Hooks

- Trace schema: 每个 span 记录 input、output、model/tool/provider、latency、cost、error、eval score。
- Eval registry: 区分 end-to-end eval、component-level eval、latency/cost benchmark。
- Error taxonomy: 维护可多选的 error tags，并定期回看是否需要合并/拆分。
- Dashboard: 同时展示 quality、failure category、latency、cost。
- Review cadence: 每轮 build 后固定抽样读 outputs/traces，防止只看指标。
- Decision log: 记录“为什么这次优化这个 component”，避免团队失忆。

## My Notes

- 这节像是 Module 4 的总 SOP：不要只做 coding agent，也要做 analyst agent，分析哪里值得改。
- 对我来说最重要的一句话是：analysis 也算进展。读 trace、标错误、设计 eval 不是“没产出”，它是在给下一轮 build 定方向。
- 这个 repo 本身也应该按这个节奏来：先整理 lesson notes，再抽 concepts，再做 labs/evals，而不是一上来改 upstream 代码。

## Open Questions

- 对数据分析 agent，build/analyze 的节奏应该怎么安排：每改一个 component 就跑 eval，还是每天/每周集中分析？
- Error taxonomy 到什么粒度最合适，才能既指导优化又不难维护？
- Custom evals 应该先覆盖最常见错误，还是先覆盖最高风险错误？
- 如何让 decision log 和 eval results 绑定，避免反复尝试已经失败的优化方向？

## Related

- [Evaluations (evals)](01-evaluations-evals.md)
- [Error analysis and prioritizing next steps](02-error-analysis-and-prioritizing-next-steps.md)
- [Component-level evaluations](04-component-level-evaluations.md)
- [Latency, cost optimization](07-latency-cost-optimization.md)
- [Agentic development loop concept card](../../concepts/agentic-development-loop.md)
- [Agentic evals](../../concepts/agentic-evals.md)

## Next Actions

- [x] 更新 Module 4 README / summary。
- [x] 增加 agentic development loop 概念卡。
- [x] 增加 glossary / open questions。
- [ ] 后续补 Module 4 quiz 复习。
- [ ] 后续补 ungraded lab: adding a component-level eval to the research workflow。
