---
course: Agentic AI
module: 1
lesson: 4
title: Benefits of Agentic AI
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/k2vehc/benefits-of-agentic-ai
watched_at: 2026-05-29
reviewed_at:
tags: [agentic-workflows, performance, parallelism, modularity, evals]
---

# Benefits of Agentic AI

## TL;DR

Agentic workflows 的最大好处是：它们能让 LLM 完成一些过去很难有效完成的任务。除此之外，它们还带来两个很实用的工程收益：可以把部分任务并行化，并且可以用模块化方式组合、替换工具和模型。

## Core Ideas

- Agentic workflow 往往比单次 prompt 直接生成更强，因为它可以把任务拆成多个步骤，让模型写、查、读、反思、修改。
- 即使底层模型不变，外面包一层 agentic workflow，也可能显著提升任务表现。
- Agentic workflow 不是一定更快；相比单次 LLM 调用通常更慢，但相比人类串行处理复杂任务，某些步骤可以并行，因此整体可能更快。
- 模块化是 agentic workflow 的重要优势：可以替换 search engine、加入 news search、换不同 LLM provider，甚至让不同步骤使用不同模型。
- 设计 agentic system 时，不只要问“用哪个模型”，还要问“workflow 怎么拆、哪些步骤并行、哪些组件可以替换”。

## Example From The Lesson

课程用 coding benchmark 说明：单纯换更强模型能提升表现，但把模型放进 agentic workflow 后，也能带来很大的性能提升。

这里的重点不是记住某个 benchmark 数字，而是记住这个设计判断：

> Model capability 和 workflow design 都会影响最终表现。强模型很重要，但好的 workflow 也可能带来巨大收益。

课程还继续用写 black holes essay 的研究任务来说明 parallelism：

1. 多个 LLM call 可以并行生成不同 search queries。
2. 多个 web search 结果可以分别选择要读取的页面。
3. 多个 web pages 可以并行 fetch。
4. 最后再把收集到的信息交给 LLM 整合成文章。

人类通常要一页一页读，但 agentic workflow 可以同时下载和处理多个信息来源。

## Three Main Benefits

| Benefit | Meaning | Design Implication |
| --- | --- | --- |
| Better performance | 多步骤 workflow 可能比单次生成更可靠、更强 | 不要只调 prompt，也要设计步骤、反思、工具使用和 eval |
| Parallelism | 某些搜索、读取、分析步骤可以同时跑 | 找出可以并发的子任务，减少整体等待时间 |
| Modularity | 工具、模型、搜索源可以替换或组合 | 把 workflow 设计成可插拔组件，而不是一整块硬编码逻辑 |

## Design Takeaways

- 不要把 agentic AI 只理解成“LLM 自己行动”。更实用的理解是：用 workflow 结构放大 LLM 的能力。
- 单次 prompt 是一个 baseline；agentic workflow 是在 baseline 上增加步骤、工具、反馈和控制。
- 并行化不是随便并发调用，而是要找到互不依赖的子任务，比如多个 search queries、多个 page fetches、多个候选方案生成。
- 模块化让优化变得更现实：某一步效果不好时，可以替换 search engine、换模型、换工具，而不必重写整个系统。
- 不同 workflow step 可能适合不同模型。便宜模型可以处理简单步骤，强模型可以处理关键推理或最终整合。

## Failure Modes

- 为了追求 agentic workflow，把一个简单任务拆得太复杂，导致 latency、cost 和 debugging burden 上升。
- 并行化太多，fetch 了大量低质量来源，反而增加噪音。
- 只看 final answer，不评估中间步骤是否选对了 query、source、tool 或 model。
- 模块化边界没设计好，导致组件之间输入输出格式不稳定。
- 每个步骤都换模型或工具，但没有 eval 判断替换是否真的变好。

## Implementation Hooks

- Baseline: 先建立 single-prompt baseline，再比较 agentic workflow 是否真的提升。
- Parallelism: 把 independent subtasks 拆出来并发执行，比如 query generation、page fetch、候选答案生成。
- Modularity: 为每个工具定义清晰 input/output contract，方便替换 search engine、retriever、LLM 或 parser。
- Evals: 不只评估最终答案，也评估 search quality、source quality、reflection usefulness 和 end-to-end cost。
- Routing: 简单步骤用便宜模型，关键步骤用更强模型，减少整体成本。
- Observability: 记录每一步用了哪个 tool/model、耗时、成本和输出质量。

## My Questions

- 对一个任务，什么时候 agentic workflow 的性能提升值得额外成本？
- 如何判断哪些步骤应该并行，哪些步骤必须串行？
- 如果替换 search engine 或 LLM provider，应该用什么 eval 来比较？
- 多模型 workflow 里，如何避免不同模型输出格式不一致？

## Related

- [Degrees of Autonomy](03-degrees-of-autonomy.md)
- [Agentic workflow benefits](../../concepts/agentic-workflow-benefits.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 agentic workflow benefits concept card。
- [ ] 后续做 lab 时，用 single-prompt baseline vs agentic workflow 做对照。
- [ ] Module 1 完成后补复习题。

