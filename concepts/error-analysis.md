# Concept: Error Analysis

## Definition

Error analysis 是系统化阅读失败样例和 traces，判断 final output 为什么失败、哪个 component 最可能负责、以及下一步最值得改哪里的方法。

它不是简单地说“系统错了”，而是把错误拆到 workflow 的具体步骤上。

## Why It Matters

Agentic workflow 往往包含 search、retrieval、tool use、code execution、LLM reasoning、final writing 等多个组件。最终结果不好时，如果只凭直觉改 prompt，很容易花很多时间却没有整体改善。

Error analysis 能让改进变成更像数据分析：

1. 收集失败样例。
2. 阅读每一步 trace/span。
3. 标注哪个 component 明显低于 human expert baseline。
4. 统计错误集中在哪里。
5. 根据错误频率和可改进性决定优先级。

## Core Questions

- 这个 final output 为什么不满意？
- 失败是从哪个 upstream component 开始的？
- 下游 component 是真的做差了，还是被坏输入拖累？
- 这个 component 的输出相比 human expert 会差多少？
- 这个错误在失败样例里出现频率高吗？
- 我有没有具体办法修这个 component？

## Trace And Span

| Term | Meaning | Example |
| --- | --- | --- |
| Trace | 一次 agent run 的完整中间执行记录 | 用户问题、search query、URLs、fetched docs、selected sources、final answer |
| Span | Trace 中某一个步骤的输出 | Search terms、SQL draft、retrieved documents、chart spec |

Trace 帮你复盘系统做了什么；span 帮你定位是哪一步开始偏离预期。

## Prioritization

优先级不只看错误频率，也看改进可行性：

| Situation | Priority |
| --- | --- |
| 错误多，而且有明确修法 | 高优先级 |
| 错误多，但暂时不知道怎么修 | 记录下来，可能需要更多研究 |
| 错误少，但修复成本很低 | 可以作为 quick win |
| 错误少，修复成本高 | 暂缓 |

## Example

Research agent 漏掉关键观点时，不一定是 final writing prompt 的问题。

可能链路：

```text
search terms -> web search results -> source selection -> web fetch -> synthesis -> final essay
```

如果 search terms 已经合理，但 search results 主要是低质量博客，那么优先改 web search engine、search parameters 或 source constraints。此时 source selection 选不到好 source，不一定是它自己的错。

## Additional Examples

### Invoice Processing

Workflow:

```text
invoice PDF -> PDF-to-text -> LLM field extraction -> database record
```

如果 due date 抽错，先看 PDF-to-text 输出。如果 OCR/parser 已经让人类也看不清 due date，那么问题在 PDF-to-text；如果文本足够清楚但 LLM 仍然抽错日期，那么问题更可能在 LLM extraction。

### Customer Email Assistant

Workflow:

```text
customer email -> query generation -> database lookup -> email drafting -> human review
```

最终 email 不满意时，可能是 LLM 查错表/字段，也可能是 database 本身有脏数据，还可能是拿到正确数据后 final drafting 写得不好。先归因，再决定是优化 query generation、数据质量，还是 final email prompt。

## Non-Mutually Exclusive Errors

一个失败样例可以有多个错误标签，所以错误比例不一定加起来等于 100%。这比强行单选更贴近真实系统，因为上游和下游可能同时有问题。

在 prioritization 时，除了统计频率，还要判断哪个错误最早发生、是否导致下游失败、以及修复它是否能带来整体改进。

## Data Analyst Translation

对数据分析 agent，可以这样做 error analysis：

| Component | Span To Inspect |
| --- | --- |
| Intent understanding | 用户问题被改写成什么分析目标 |
| Schema lookup | 找到了哪些表、字段、指标定义 |
| SQL generation | 生成的 SQL 是否符合口径和粒度 |
| SQL execution | 是否报错，结果是否异常 |
| Result interpretation | 是否过度解读或漏掉关键变化 |
| Visualization | 图表类型、字段映射、标题和坐标轴是否合适 |
| Final answer | 结论是否有 evidence，是否说明限制 |

## Failure Modes

- 只看 final output，不保存 intermediate traces。
- 把上游坏输入导致的失败算到下游 component。
- 只看几个印象深刻的例子，不统计错误频率。
- 只看错误频率，不看修复成本和可行方案。
- 没有 human expert baseline，导致不知道什么算 component output 差。
- 强行让错误标签互斥，漏掉同一个失败样例中的多个真实问题。

## Related Notes

- [Component-level eval](component-level-eval.md)
- [More error analysis examples](../notes/module-04-evals-and-optimization/03-more-error-analysis-examples.md)
- [Error analysis and prioritizing next steps](../notes/module-04-evals-and-optimization/02-error-analysis-and-prioritizing-next-steps.md)
- [Evaluations (evals)](../notes/module-04-evals-and-optimization/01-evaluations-evals.md)
- [Agentic evals](agentic-evals.md)
