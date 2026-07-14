---
course: Agentic AI
module: 4
lesson: 7
title: "Latency, cost optimization"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/154qpa/latency%2C-cost-optimization
watched_at: 2026-07-14
reviewed_at:
tags: [latency, cost-optimization, benchmarking, parallelism, model-selection, observability]
---

# Latency, Cost Optimization

## TL;DR

Agentic workflow 的第一优先级通常是 output quality；当系统已经真的有用、开始进入生产或用户量上来之后，再系统优化 latency 和 cost。优化不要凭感觉，要先按 component 记录耗时和成本，找出最值得优化的瓶颈，再考虑 parallelism、更快/更便宜的模型、不同 provider 或替换昂贵组件。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- 先把质量做出来，再优化 latency/cost；否则容易把还没工作的系统优化得很快、很便宜，但没有价值。
- 当 cost 或 latency 变成问题时，先 benchmark 每个 workflow step。
- Latency 优化要看 timeline：哪个 step 最慢，哪个 step 可以并行，哪个 LLM call 可以换更快模型或 provider。
- Cost 优化要看 per-step cost：LLM token cost、API call cost、compute/server cost、PDF-to-text 或其他工具成本。
- Benchmark 结果会告诉你哪些 component 值得优化，哪些 component 对总成本/总延迟贡献太小，不值得花时间。
- 优化后仍要保留质量 eval，避免为了速度或成本牺牲核心输出质量。

## Lecture Map

1. 开场观点：输出质量通常最难，cost/latency 往往可以晚一点优化。
2. 当产品真的被使用、成本或等待时间成为问题时，再进入优化阶段。
3. Latency 优化：给每个 step 计时，看整体 timeline。
4. 可尝试的 latency 修法：并行执行独立步骤、换更快/更小模型、尝试更快 LLM provider。
5. Cost 优化：计算每个 step 的成本，包括 token、API call、compute、外部服务。
6. 用 per-step benchmark 决定优化优先级，避免优化不重要的小项。
7. 过渡到 Module 4 最后一节 development process summary。

## Latency Optimization

Latency 是用户等待完整 workflow 结果的时间。对 agentic workflow 来说，它通常由多个 step 累加而成：

```text
generate search terms
  -> web search
  -> web fetch / retrieval
  -> synthesis
  -> final answer
```

优化前先做 timing breakdown：

| Step | What To Measure | Possible Fix |
| --- | --- | --- |
| LLM search-term generation | seconds per call | smaller/faster model, faster provider, prompt simplification |
| Web search | API response time | provider switch, narrower query, caching |
| Web fetch / retrieval | time per source | parallel fetch, fewer sources, timeout |
| Final writing/synthesis | seconds and output tokens | faster model, shorter context, split or simplify output |

核心不是“所有步骤都优化”，而是看 timeline 里谁最慢、谁在 critical path 上。

## Cost Optimization

Cost 也要拆到 step 级别：

| Cost Type | Example |
| --- | --- |
| Token cost | LLM input/output tokens |
| API call cost | web search、retrieval、PDF-to-text、external service |
| Compute cost | code execution、server capacity、worker runtime |
| Storage / bandwidth | logs、artifacts、retrieved documents |

课程里的方法很朴素但有效：给每个 step 估算平均 cost，再看哪个 component 是最大 contributor。很多时候 benchmark 会告诉你：某些 component 成本很小，不值得优化；真正的大头可能是某个外部 API、某个长上下文 LLM call，或某个高频工具调用。

## Optimization Menu

Latency:

- 对互不依赖的 steps 做 parallelism，例如多个 web fetch 同时执行。
- 给慢的 LLM step 尝试更小、更快、但质量仍足够的模型。
- 尝试不同 LLM provider，因为同一类模型的 token serving speed 可能差异很大。
- 减少不必要的 context、输出长度或工具调用。

Cost:

- 找出最贵 step，而不是平均用力。
- 减少 token 数：缩短 prompt、压缩 retrieved context、限制 output length。
- 对低风险/低难度 step 使用更便宜的模型。
- 替换昂贵 API，或在质量允许时减少调用次数。
- Cache 可复用结果，例如 schema、静态文档 retrieval、重复 search/fetch。

## Data Analyst Translation

如果我做数据分析 agent，可以这样记录每次 run：

| Step | Latency | Cost | Quality Check |
| --- | --- | --- | --- |
| Schema retrieval | ms/sec | vector DB / search cost | 是否找对表字段 |
| SQL generation | LLM seconds | token cost | SQL 是否可执行、口径是否正确 |
| SQL execution | query runtime | warehouse cost | 结果是否可信 |
| Chart generation | Python/runtime | compute cost | 图表 spec 是否合理 |
| Final explanation | LLM seconds | token cost | 是否基于数据、无过度解读 |

这样才能回答一个真实工程问题：为了让 agent 更快/更便宜，我应该先优化 schema retrieval、SQL 生成、数据库查询，还是最终解释？

## Design Takeaways

- Quality-first 不是忽略成本，而是避免过早优化错误目标。
- Cost/latency 优化也需要 observability：没有 per-step timing/cost，就不知道该优化哪里。
- Parallelism 是 agentic workflow 的天然优势，但只适合互不依赖的步骤。
- 更小/更快/更便宜的模型可以用于局部 component，但必须用 component-level eval 保证质量仍够用。
- 优化预算要看实际贡献：不重要的 100ms 或 0.01 cent 不值得复杂化系统。

## Failure Modes

- 还没证明 workflow 有用，就过早追求便宜/快速。
- 只看总延迟，不看每个 component 的 timeline，导致优化方向错误。
- 为了省 token 删掉关键上下文，使质量下降。
- 把所有步骤都换成便宜模型，没有做 component-level eval。
- 并行化有依赖关系的步骤，造成结果不稳定或更难 debug。
- 只优化平均 latency，不关注 p95/p99 等用户真正感受到的长尾等待。

## Implementation Hooks

- Trace: 每个 span 记录 `start_time`、`end_time`、`duration_ms`、`provider`、`model`、`input_tokens`、`output_tokens`、`api_cost`。
- Dashboard: 同时看 quality score、latency、cost，不单独追一个指标。
- Config: 把 model/provider/tool timeout/cache 策略配置化，方便 A/B 和回滚。
- Eval: 换更快/更便宜组件后，先跑 component-level eval，再跑 end-to-end eval。
- Budget: 为 workflow 设定可接受的 max latency 和 max cost per run。

## My Notes

- 这节跟数据分析 agent 很贴：慢不一定是 LLM 慢，可能是 database query、schema retrieval、Python 执行或图表渲染慢。
- 我应该从一开始就记录 latency/cost trace，但不急着优化；先让系统能稳定产出正确分析。
- 真正进入产品化时，应该有一张表能回答：每个 step 花了多少时间、多少钱、贡献了多少质量。

## Open Questions

- 数据分析 agent 应该记录 average latency，还是同时记录 p50/p95/p99？
- SQL warehouse cost 如何和 LLM token cost 放在同一张 cost dashboard 里？
- 哪些步骤可以安全并行，哪些必须按顺序等待上游结果？
- 如果小模型让某个 component 成本下降 80%，但质量下降 2%，业务上是否可以接受？

## Related

- [How to address problems you identify](06-how-to-address-problems-you-identify.md)
- [Component-level evaluations](04-component-level-evaluations.md)
- [Latency/cost optimization concept card](../../concepts/latency-cost-optimization.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 4 README / summary。
- [x] 增加 latency/cost optimization 概念卡。
- [x] 增加 glossary / open questions。
- [x] 下一节继续 development process summary。
