# Concept: Latency/Cost Optimization

## Definition

Latency/cost optimization 是在 agentic workflow 已经能产出足够高质量结果后，系统化降低等待时间和运行成本的工程过程。

它不是“把所有东西都换成便宜模型”，而是先测量每个 component 的耗时和成本，再把优化精力放到真正的大头上。

## Why It Matters

Agentic workflow 通常会串联多个 LLM calls、tools、retrieval、web/API calls 和 code execution。系统越有用、用户越多，成本和延迟越容易变成产品问题。

但过早优化也危险：如果 output quality 还不够好，系统再快再便宜也没有意义。因此这类优化通常排在 quality eval 和 error analysis 之后。

## Core Loop

1. 先确认 workflow 的 output quality 达到可用水平。
2. 给每个 span 记录 latency 和 cost。
3. 汇总 per-step timeline 和 per-step cost。
4. 找出最大 contributor。
5. 对该 component 尝试 parallelism、model/provider swap、token reduction、caching 或 tool replacement。
6. 跑 component-level eval，确认质量没有明显下降。
7. 跑 end-to-end eval，确认整体用户体验变好。

这通常属于 agentic development loop 的后期：先通过 eval/error analysis 证明系统有用，再用 latency/cost measurement 让系统更适合生产使用。

## What To Measure

| Dimension | Examples |
| --- | --- |
| Latency | step duration, total duration, p50/p95/p99, queue time |
| LLM cost | input tokens, output tokens, model/provider price |
| Tool cost | search API call, PDF-to-text, vector DB query, external service |
| Compute cost | code execution runtime, server/worker capacity, warehouse query cost |
| Quality | component score, end-to-end score, human approval rate |

## Optimization Options

| Problem | Possible Fix |
| --- | --- |
| Independent fetch/search calls are slow | Run them in parallel |
| LLM step is slow | Try faster provider, smaller model, shorter prompt/context |
| LLM step is expensive | Reduce tokens, route easy cases to cheaper model |
| External API is expensive | Cache, batch, reduce calls, replace provider |
| Compute step is slow or costly | Optimize query/code, set resource limits, precompute |

## Data Analyst Translation

对数据分析 agent，不要只看“总共等了多久”。应该拆开：

- schema retrieval 花了多久、找得准不准。
- SQL generation 花了多少 tokens，SQL 是否通过 eval。
- SQL execution 是不是 warehouse cost 大头。
- Python/chart step 是否重复计算。
- final explanation 是否用了过多 context 或输出太长。

然后再决定先优化哪一步。

## Failure Modes

- 质量还没达到可用水平，就过早做成本优化。
- 只看总成本/总耗时，不记录 per-step breakdown。
- 为了降低 token 数，删掉关键上下文，导致答案质量下降。
- 并行化有依赖关系的步骤，使 workflow 行为不稳定。
- 把每个 component 都优化一遍，而不是聚焦最大 contributor。
- 只看平均 latency，不看用户会感受到的长尾延迟。

## Related Notes

- [Latency, cost optimization](../notes/module-04-evals-and-optimization/07-latency-cost-optimization.md)
- [How to address problems you identify](../notes/module-04-evals-and-optimization/06-how-to-address-problems-you-identify.md)
- [Component-level eval](component-level-eval.md)
- [Agentic evals](agentic-evals.md)
- [Agentic development loop](agentic-development-loop.md)
- [Task decomposition](task-decomposition.md)
