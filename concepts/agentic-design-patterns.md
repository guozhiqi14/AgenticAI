# Concept: Agentic Design Patterns

## Definition

Agentic design patterns 是把 LLM、tools、APIs、retrieval、code execution、human review 等 building blocks 组合成 agentic workflow 的常见结构。

## Why It Matters

Design patterns 给了我们设计 agentic workflow 的“结构语言”。它们让我们不只是问“用什么模型”，而是问“系统应该如何检查自己、调用工具、规划步骤、分工协作，并如何评估这些行为”。

## Four Key Patterns

| Pattern | Main Idea | Best For | Watch Out |
| --- | --- | --- | --- |
| Reflection | 生成后检查、批评、修正 | 代码、写作、答案草稿、可检查输出 | 没有外部信号时可能自我确认 |
| Tool use | LLM 请求调用外部函数或工具 | 搜索、计算、数据库、email、calendar、业务系统 | 工具选择、参数错误、权限、外部副作用 |
| Planning | 模型运行时生成 plan 并逐步执行 | 开放任务、多工具组合、动态流程 | 难控制、难 eval、可能乱规划 |
| Multi-agent collaboration | 多角色 agent 分工协作 | 复杂任务、多视角任务、review/edit 流程 | 协调成本高、debug 困难 |

## Design Questions

- 这个任务的主要瓶颈是质量、信息、计算、规划，还是协作？
- 是否有外部反馈可以支持 reflection？
- 哪些能力应该做成 tool，而不是让 LLM 纯生成？
- Tool 应该由 developer hard-code 调用，还是交给 LLM 自主选择？
- 步骤是否需要模型动态 planning，还是工程师 hard-code 更可靠？
- 多个 agent 的角色边界是否清晰？
- 每个 pattern 的效果如何通过 eval 验证？

## Practical Rule

先用最简单的 pattern 解决明确问题。不要为了“更 agentic”而堆叠 reflection、tool use、planning 和 multi-agent。每增加一个 pattern，都要对应一个具体 failure mode 或能力缺口。

## Related Notes

- [Agentic Design Patterns](../notes/module-01-introduction-to-agentic-workflows/08-agentic-design-patterns.md)
- [Autonomy spectrum](autonomy-spectrum.md)
- [Task decomposition](task-decomposition.md)
- [Tool use](tool-use.md)
- [Planning pattern](planning-pattern.md)
- [Agentic evals](agentic-evals.md)
