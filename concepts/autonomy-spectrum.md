# Concept: Autonomy Spectrum

## Definition

Autonomy spectrum 描述的是：一个 agentic system 到底把多少决策权交给 LLM，而不是固定在工程师预先写好的 workflow 里。

这不是二元分类。一个系统可以有一点 agentic，也可以中度 agentic，或者高度 agentic，取决于它被允许做哪些决策。

## The Core Question

设计任何 agentic workflow 时，先问：

> 哪些决策固定在代码里，哪些决策允许 LLM 在运行时做？

## Levels

| Level | Description | Good For | Main Risk |
| --- | --- | --- | --- |
| Less autonomous | Steps and tools are mostly predetermined. | Repeatable workflows, production reliability, lower risk tasks. | Too rigid for ambiguous tasks. |
| Semi-autonomous | The LLM chooses among predefined options or tools. | Research, triage, multi-step analysis, routing. | Needs guardrails and action evals. |
| Highly autonomous | The LLM can plan many steps, loop, and sometimes create tools. | Open-ended tasks and research prototypes. | Harder to control, debug, and evaluate. |

## Design Questions

- 这个任务有没有已知的最佳步骤顺序？
- 如果 agent 做错决策，结果是否可逆？
- 工具调用是否昂贵、涉及隐私，或者会对外部世界产生影响？
- agent 的 action choice 是否能被记录和评估？
- 灵活性是否值得额外的 latency、cost 和 unpredictability？

## Practical Rule

使用“能很好解决问题的最低 autonomy level”。

当任务需要探索、分支、判断或适应时，提高 autonomy。当任务需要可预测、合规、成本控制或可重复时，降低 autonomy。

Planning pattern 是把系统推向 highly autonomous 的常见方式：developer 不再固定每一步，而是让 LLM 在 runtime 生成 plan。使用它之前要确认 plan/action choice 能被记录、评估和必要时拦截。

## Related Notes

- [Degrees of Autonomy](../notes/module-01-introduction-to-agentic-workflows/03-degrees-of-autonomy.md)
- [Planning pattern](planning-pattern.md)
