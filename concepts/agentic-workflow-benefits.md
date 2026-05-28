# Concept: Agentic Workflow Benefits

## Definition

Agentic workflow benefits 指的是：通过把 LLM 放进多步骤、可调用工具、可并行、可替换组件的 workflow 中，获得比单次 prompt 更强或更可控的系统表现。

## Why It Matters

很多时候，提升 AI 应用效果不只是换一个更强模型。Workflow design 本身也会显著影响表现、速度、成本和可维护性。

## Main Benefits

| Benefit | What It Gives You | What To Watch |
| --- | --- | --- |
| Performance | 多步骤推理、工具使用、反思和修改带来更好结果 | 需要 eval 证明真的提升 |
| Parallelism | 独立子任务可以并发，适合搜索、读取、候选生成 | 可能增加噪音和成本 |
| Modularity | 可以替换工具、模型、数据源或 parser | 需要稳定的 input/output contract |

## Design Questions

- 单次 prompt 的 baseline 表现如何？
- 哪些步骤可以独立并行？
- 哪些组件未来可能需要替换？
- 每一步是否有清晰的输入、输出和评价标准？
- 增加 workflow complexity 后，性能提升是否值得 latency 和 cost？

## Practical Rule

先建立简单 baseline，再逐步加入 agentic workflow。每增加一个步骤、工具或并行分支，都要问：它是否带来了可测量的收益？

## Related Notes

- [Benefits of Agentic AI](../notes/module-01-introduction-to-agentic-workflows/04-benefits-of-agentic-ai.md)
- [Autonomy spectrum](autonomy-spectrum.md)

