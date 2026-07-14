# Concept: Agentic Development Loop

## Definition

Agentic development loop 是构建 agentic workflow 时在 build 和 analyze 之间来回切换的开发循环。

Build 是写代码、改 prompt、换模型、调工具；analyze 是读 outputs/traces、做 eval、error analysis 和 component attribution。两者缺一不可。

## Why It Matters

Agentic workflow 的失败模式很难完全预判。只靠预先设计，很容易在错误方向上做很多工程。更稳的方式是快速做出 end-to-end prototype，然后用真实 outputs 和 traces 让系统告诉你哪里最值得修。

经验不足的团队常见问题是 build 太多、analyze 太少：一直改 prompt、换模型、加工具，但没有系统回答“为什么改这里”。

## Core Loop

```text
build quick prototype
  -> inspect outputs/traces
  -> create small eval set
  -> run end-to-end eval
  -> do error analysis
  -> build component-level evals
  -> improve selected components
  -> re-run end-to-end eval
  -> repeat
```

这个 loop 不是瀑布式流程；成熟系统也会在 end-to-end tuning、component tuning、error analysis、latency/cost optimization 之间来回切换。

## Maturity Ladder

| Maturity | Main Activity | Output |
| --- | --- | --- |
| Prototype | 快速跑通 end-to-end | 能观察真实输出 |
| Observation | 手动读 outputs/traces | 初步 failure mode |
| End-to-end eval | 10-20 examples + metric | 整体质量趋势 |
| Error analysis | 失败样例归因 | 优先级列表 |
| Component eval | 局部 eval + comparison | 可控优化单点 |
| Production tuning | monitoring + cost/latency | 质量、速度、成本平衡 |

## Data Analyst Translation

对数据分析 agent，这个 loop 可以落成：

- Build: schema retrieval、SQL generation、SQL execution、chart generation、final explanation。
- Analyze: 看每个真实问题的 trace，判断错在 schema、SQL、数据、图表还是解释。
- Eval: 先做 end-to-end 正确性/有用性评估，再给最常错的 component 做局部 eval。
- Optimize: 只优化最值得修的 component，然后回到 end-to-end eval。

## Practical Rule

每次准备继续 build 前，先问：

- 我是根据哪个 output、trace、eval 或 error analysis 决定改这里的？
- 这个改动会用什么 eval 证明真的变好？
- 如果它让局部变好，什么时候回到 end-to-end eval？

如果回答不上来，可能是在随机试错。

## Failure Modes

- 只 build，不 analyze。
- 只看 final answer，不读 traces。
- eval set 太晚出现，导致改动无法比较。
- error taxonomy 太粗，不能指导下一步。
- component eval 变好后，不做 end-to-end regression。
- 迷信通用 monitoring tool，却没有 custom eval 捕捉业务 failure mode。

## Related Notes

- [Development process summary](../notes/module-04-evals-and-optimization/08-development-process-summary.md)
- [Agentic evals](agentic-evals.md)
- [Error analysis](error-analysis.md)
- [Component-level eval](component-level-eval.md)
- [Latency/cost optimization](latency-cost-optimization.md)
