# Concept: Agentic Application Fit

## Definition

Agentic application fit 指的是：一个任务场景是否适合用 agentic workflow 来实现，以及实现难度大概在哪个区间。

## Why It Matters

不是所有任务都应该做成高自主性的 agent。好的应用选择能让系统更可靠、更容易评估，也更容易上线。糟糕的应用选择会把 workflow 复杂度、工具失败、多模态输入、不可逆动作和 eval 难度全部叠在一起。

## Easier Fit

更适合先做的任务通常有这些特征：

- 已经存在 SOP 或标准业务流程。
- 输入主要是文本，或者能可靠转换成文本。
- 输出字段或回复格式明确。
- 需要调用的工具有稳定 API。
- 每一步可以单独观察和评估。
- 高风险动作可以放到 human review 后再执行。

## Harder Fit

更难的任务通常有这些特征：

- 用户请求开放，步骤无法提前确定。
- 需要 runtime planning 和多轮工具选择。
- 输入包含复杂视觉、音频、网页、表格或 GUI 状态。
- 需要操作浏览器或桌面软件，而不是稳定 API。
- 行动不可逆，或者对外部系统有直接影响。
- 成功标准模糊，难以写 eval。

## Design Questions

- 这个任务有没有明确的 SOP？
- 输入是否能稳定转成 LLM 可处理的文本？
- 需要哪些工具？这些工具是 API 还是 GUI/browser？
- 哪些动作需要 human approval？
- 失败后能不能定位到具体步骤？
- 是否能分别评估 extraction、planning、tool use 和 final output？

## Practical Rule

先从流程清楚、文本为主、工具稳定、可人工审核的任务开始。等 workflow、eval 和 observability 都成熟后，再逐步提高 autonomy 和输入复杂度。

## Related Notes

- [Agentic AI Applications](../notes/module-01-introduction-to-agentic-workflows/05-agentic-ai-applications.md)
- [Autonomy spectrum](autonomy-spectrum.md)

