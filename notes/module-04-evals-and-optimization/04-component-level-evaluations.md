---
course: Agentic AI
module: 4
lesson: 4
title: "Component-level evaluations"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/fwxyds/component-level-evaluations
watched_at: 2026-07-09
reviewed_at:
tags: [component-level-eval, end-to-end-eval, web-search, information-retrieval, gold-standard, evals]
---

# Component-Level Evaluations

## TL;DR

End-to-end eval 能告诉你整个 agentic workflow 有没有变好，但当 error analysis 已经定位到某个 component 时，只靠 end-to-end eval 会又贵又吵。Component-level eval 单独评估一个组件，让你更快、更清楚地判断这个组件是否真的被改好了；但最后仍然要回到 end-to-end eval，确认局部改进能转化成整体质量提升。

## Core Ideas

- Component-level eval 评估单个 component，而不是整个 workflow 的最终输出。
- 它适合在 error analysis 已经指出某个 component 是瓶颈后使用。
- 只跑 end-to-end eval 可能成本高、速度慢，而且其他 component 的随机性会掩盖小改进。
- Component-level eval 给局部优化更清晰的 signal。
- 对 web search component，可以准备 gold standard web resources，检查搜索结果是否覆盖专家认为应该找到的来源。
- 信息检索里常见的指标，如 precision、recall、F1，可以用于衡量搜索结果和 gold standard 的重合程度。
- Component-level eval 适合调参数、换工具、换模型、改 prompt 时快速比较。
- 局部指标变好不等于最终系统一定变好；完成局部优化后还要跑 end-to-end eval。

## Lecture Map

1. 承接前几节：research agent 漏掉 key points，error analysis 可能发现 web search 是主要问题。
2. 说明只用 end-to-end eval 的问题：贵、慢、复杂组件的噪声会遮住 web search 的小幅改进。
3. 提出 component-level eval：单独评估 web search 结果质量。
4. 用 gold standard web resources 作为参考：专家标注某个 query 应该找到哪些权威来源。
5. 用信息检索指标衡量 search outputs 和 gold standard 的重合程度。
6. 用这个局部指标快速比较 search engine、result count、date range 等参数。
7. 最后再跑 end-to-end eval，验证局部优化是否真的提升整体 workflow。

## Example: Web Search Component

问题背景：

```text
user prompt -> search-term generation -> web search -> source selection -> fetch -> synthesis -> final essay
```

如果 error analysis 发现 research agent 漏掉 key points 的主要原因是 web search 结果质量差，那么下一步不一定要每次都跑完整 research agent。可以先做一个只评估 web search 的 component-level eval。

做法：

- 准备一组 representative queries。
- 让专家为每个 query 标注 gold standard web resources。
- 对每个 query 跑当前 web search component。
- 比较返回 URL 和 gold standard resources 的 overlap。
- 用 precision / recall / F1 或类似指标衡量搜索质量。

可以比较的变化包括：

- 换 web search engine。
- 改搜索参数或 hyperparameters。
- 调整返回结果数量。
- 限定 date range。
- 增加 domain/source constraints。

## Why Not Only End-To-End Eval?

End-to-end eval 仍然必要，但不适合所有调参场景：

| Problem | Why Component-Level Eval Helps |
| --- | --- |
| End-to-end workflow 成本高 | 单独跑一个 component 更快、更便宜 |
| 其他组件有随机性 | 局部 eval 减少无关噪声 |
| 小改进难以观察 | 局部指标 signal 更清楚 |
| 多团队协作 | 每个团队可以优化自己的 component metric |
| 调参频繁 | 更适合快速比较多个候选方案 |

可以把它理解成数据 pipeline 调优：如果你怀疑是特征生成有问题，就先单独评估特征质量，不必每次都训练完整模型再看最终业务指标。

## Design Takeaways

- 先用 end-to-end eval 和 error analysis 找瓶颈，再用 component-level eval 优化瓶颈。
- Component-level eval 的输入输出要尽量固定，减少其他组件带来的噪声。
- Component metric 要贴近该组件对整体任务的贡献，而不是随便找一个好算的指标。
- 局部优化完成后必须回归 end-to-end eval，避免局部指标变好但整体体验没变好。
- 对数据分析 agent，适合单独做 eval 的 component 包括 schema retrieval、SQL generation、SQL execution validation、chart spec generation、final explanation。

## Failure Modes

- 只优化 component metric，忘了验证 end-to-end quality。
- Component metric 选错，导致局部指标提升但最终答案没有改善。
- Gold standard resources 或 labels 太少，导致 eval 只覆盖少数熟悉 case。
- Component 输入没有固定，导致评估结果仍然被上游变化污染。
- 过早给每个小步骤都建 eval，维护成本超过收益。

## Implementation Hooks

- Prompting: 对需要评估的 component 固定输入格式，方便重复运行。
- Tools: 保存 component inputs/outputs，比如 query、returned URLs、SQL、chart spec。
- Workflow: 支持只运行单个 component，而不是每次跑完整 workflow。
- Memory/state: 记录 component version、tool/provider、参数、eval score。
- Evals: 使用 gold standard labels、objective metrics 或 LLM-as-a-judge，取决于 component 的输出类型。
- Human-in-the-loop: 由专家先标注 gold standard，再定期抽查 component metric 是否符合业务判断。

## My Notes

- 这节很像数据科学里的 “unit test for pipeline component”。整体模型分数重要，但调一个特征、一个 SQL parser 或一个检索模块时，局部指标会快很多。
- 对我的数据分析 agent，最值得先做 component-level eval 的可能是 `SQL generation` 和 `schema/metric retrieval`，因为它们错了会直接污染后面的解释和图表。
- 但要小心 Goodhart：SQL 文本看起来对，不代表业务口径对；component-level eval 必须和 end-to-end eval 互相校验。

## Open Questions

- 数据分析 agent 的第一批 component-level eval 应该先评 `SQL generation`，还是先评 `schema retrieval`？
- 对 web/search/retrieval component，gold standard resources 应该由谁标注，标多少才够用？
- Component-level metric 多高才值得回到 end-to-end eval 做整体确认？
- 如何设计 component inputs，避免上游 prompt 改动让局部 eval 不可比？

## Related

- [More error analysis examples](03-more-error-analysis-examples.md)
- [Component-level eval concept card](../../concepts/component-level-eval.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Error analysis](../../concepts/error-analysis.md)
- [Module 4 summary](../../review/module-04-summary.md)

## Next Actions

- [x] 抽取 `component-level-eval` concept card。
- [x] 更新 Module 4 README / summary。
- [ ] 下一节做 ungraded lab: adding a component-level eval to the research workflow。
