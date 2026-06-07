# Concept: Task Decomposition

## Definition

Task decomposition 是把一个复杂任务拆成 discrete steps，并把每一步映射到可执行的 building block，例如 LLM、API、function call、database query、RAG、code execution 或 human review。

## Why It Matters

Agentic workflow 的质量很大程度取决于任务拆得好不好。拆得好，系统更容易实现、调试、评估和迭代；拆得不好，LLM 会在一个模糊步骤里承担太多责任，失败也很难定位。

## Core Heuristic

对每个步骤问：

1. 这一步能不能由 LLM、短代码、function call、API 或现有 tool 完成？
2. 如果不能，人类会怎么做？能不能继续拆得更小？

## Decomposition Pattern

1. Start with direct generation or the simplest workflow.
2. Compare the result with what you want.
3. Identify the weak step.
4. Split that step into smaller steps.
5. Add tools, APIs, retrieval, code execution, or human review where useful.
6. Evaluate and iterate.

## Building Blocks

| Building Block | Useful For |
| --- | --- |
| LLM | 生成、抽取、分类、解释、决定下一步 |
| Multimodal model | 处理图片、音频、视觉输入 |
| Specialized AI model | PDF conversion、OCR、speech、image analysis |
| API/tool | 搜索、发送邮件、查询天气、查日历、业务系统操作 |
| Database query | 查订单、库存、客户记录 |
| RAG/retrieval | 从大量文本中找相关内容 |
| Code execution | 计算、转换、分析、自动化 |
| Human review | 审核高风险或对外动作 |

## Design Questions

- 人类会如何完成这个任务？
- 哪些步骤可以自动化，哪些步骤需要人审核？
- 每一步的输入和输出是什么？
- 每一步失败时能否单独观察和重试？
- 哪一步最影响最终质量？
- 这个步骤是太粗、太细，还是刚好？

## Practical Rule

先做最简单可运行 workflow，再根据失败点迭代拆解。不要一开始就设计过度复杂的 agent，也不要把所有责任塞进一个 LLM call。

## Related Notes

- [Task Decomposition lesson](../notes/module-01-introduction-to-agentic-workflows/06-task-decomposition-identifying-the-steps-in-a-workflow.md)
- [Agentic application fit](agentic-application-fit.md)

