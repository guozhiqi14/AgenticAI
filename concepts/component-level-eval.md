# Concept: Component-Level Eval

## Definition

Component-level eval 是只评估 agentic workflow 中某一个 component 的 eval，例如 web search、schema retrieval、SQL generation、field extraction、source selection 或 final drafting。

它和 end-to-end eval 的区别是：end-to-end eval 看整个系统最终结果，component-level eval 看某一步是否做好了自己的工作。

## Why It Matters

当 error analysis 已经定位到某个 component 是瓶颈时，只跑 end-to-end eval 可能效率很低：

- 完整 workflow 更贵、更慢。
- 其他 component 的随机性会引入噪声。
- 局部小改进可能被整体波动掩盖。
- 多团队协作时，每个团队需要一个自己能直接优化的 metric。

Component-level eval 给局部优化一个更清晰、更快的 feedback signal。

## Core Loop

1. 用 end-to-end eval 和 error analysis 找到问题 component。
2. 固定该 component 的输入样例。
3. 为这些样例准备 gold standard、rubric 或 objective checks。
4. 单独运行这个 component。
5. 比较不同 prompt / model / tool / parameter 的 component score。
6. 局部优化完成后，再跑 end-to-end eval 验证整体是否变好。

## Example: Web Search

如果 research agent 的瓶颈是 web search，可以做：

- 准备一组 search queries。
- 让专家标注每个 query 应该找到的 gold standard web resources。
- 跑不同 search engine 或参数组合。
- 比较 returned URLs 和 gold standard resources 的 overlap。
- 用 precision、recall、F1 或类似指标衡量搜索质量。

这个 eval 不需要每次跑完整 research agent，就能快速判断 web search component 是否变好。

## Data Analyst Translation

对数据分析 agent，component-level eval 可以这样拆：

| Component | Possible Eval |
| --- | --- |
| Schema retrieval | 给定用户问题，是否找对表、字段、指标定义 |
| SQL generation | SQL 是否符合口径、粒度、过滤条件，并能执行 |
| SQL result validation | 结果是否和 ground truth / known answer 一致 |
| Chart spec generation | 图表类型、字段映射、标题、坐标轴是否合理 |
| Final explanation | 是否基于数据、说明限制、避免过度解读 |

## Design Questions

- 这个 component 是不是 error analysis 指出的主要瓶颈？
- Component 的输入能不能固定，避免上游变化污染评估？
- Component 的输出能不能客观评估？如果不能，rubric 是否足够明确？
- 局部 metric 是否真的和 end-to-end quality 有关系？
- 局部调优后，什么时候回到 end-to-end eval？

## Tradeoffs

| Choice | Benefit | Risk |
| --- | --- | --- |
| End-to-end eval only | 直接评估用户最终体验 | 贵、慢、信号噪声大，难定位组件 |
| Component-level eval | 快、便宜、信号清楚，方便局部调优 | 局部指标可能和整体质量脱节 |
| 两者结合 | 既能快速迭代，又能最终验证 | 需要维护多层 eval |

## Failure Modes

- 为没有明显瓶颈的 component 过早建立 eval。
- Component metric 变好，但 end-to-end output 没变好。
- Gold standard 太窄，导致只优化少数样例。
- Component 输入不固定，导致每次评估不可比。
- 只看 component eval，不再做整体回归测试。

## Related Notes

- [Component-level evaluations](../notes/module-04-evals-and-optimization/04-component-level-evaluations.md)
- [More error analysis examples](../notes/module-04-evals-and-optimization/03-more-error-analysis-examples.md)
- [Agentic evals](agentic-evals.md)
- [Error analysis](error-analysis.md)
