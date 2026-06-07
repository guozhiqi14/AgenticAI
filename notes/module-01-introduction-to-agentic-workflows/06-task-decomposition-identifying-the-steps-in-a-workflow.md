---
course: Agentic AI
module: 1
lesson: 6
title: "Task Decomposition: Identifying the Steps in a Workflow"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/moivygo8/task-decomposition%3A-identifying-the-steps-in-a-workflow
watched_at: 2026-05-29
reviewed_at:
tags: [task-decomposition, workflow-design, building-blocks, iteration, evals]
---

# Task Decomposition: Identifying the Steps in a Workflow

## TL;DR

Task decomposition 是把一个复杂任务拆成一串可执行、可观察、可评估的小步骤。拆解时的核心问题是：每一步能不能由 LLM、短代码、function call、API 或其他 tool 来完成？如果不能，就继续拆得更小。

## Core Ideas

- 不要只问“LLM 能不能直接完成这个任务”，而要问“这个任务如果由人完成，会拆成哪些步骤？”
- 一个好的 agentic workflow 通常来自反复拆解：先做简单版本，发现效果不够，再把表现不好的步骤继续拆细。
- 每个 discrete step 都应该能映射到某种 building block：LLM、multimodal model、specialized AI model、API、database query、RAG、code execution 等。
- Task decomposition 不是一次性设计完成的；它需要结合输出质量和 eval 结果不断迭代。
- 下一节 evals 会接上这个问题：拆完 workflow 后，如何评估每个步骤和整体表现。

## Main Heuristic

拆解每一步时，持续问这两个问题：

1. 这一步能不能由 LLM、短代码、function call、API 或现有 tool 完成？
2. 如果不能，人类会怎么完成这一步？能不能继续拆成更小、更容易自动化的子步骤？

这两个问题是这节课最重要的方法论。

## Examples From The Lesson

### Research Agent

目标：写一篇关于 topic X 的深度文章。

最简单的 baseline 是直接让 LLM 生成文章，但这种方式可能只覆盖表层内容。

第一版拆解：

1. Write an outline.
2. Generate/search web queries.
3. Use search results to write the essay.

如果结果仍然 disjointed，不够连贯，可以继续拆解第三步：

1. Write first draft.
2. Critique or identify parts that need revision.
3. Revise the draft.

这里的关键是：当某一步表现不好时，不一定要重写整个系统，可以只把那一步拆得更细。

### Customer Order Inquiry

目标：回复客户关于订单的基础问题。

可能拆成：

1. Extract key information: 客户是谁、订单号是什么、买了什么。
2. Query customer/order records: 调用数据库查询相关订单。
3. Draft or send response: 根据订单记录生成回复，必要时调用 email API 或进入 human review。

这类任务适合拆解，因为每一步都有相对明确的输入、输出和工具。

### Invoice Processing

目标：从 invoice 中提取字段并保存数据库。

在 PDF 已转换为文本后，可以拆成：

1. Extract required information: biller、address、due date、amount due 等。
2. Validate and save: 检查提取结果，调用数据库更新记录。

这个例子体现了一个实用模式：先用 specialized tool 做格式转换，再用 LLM 做理解和结构化抽取，最后用 API 写入系统。

## Building Blocks

构建 agentic workflow 时，可以把可用能力看成 building blocks：

- LLM / multimodal model: 生成文本、理解输入、抽取信息、决定调用什么。
- Specialized AI model: PDF to text、text-to-speech、image analysis 等。
- APIs: web search、weather、email、calendar、business systems。
- Retrieval / RAG: 从数据库或大文本集合中找相关信息。
- Code execution: 让 LLM 写代码并运行，用来做计算、转换、分析或自动化。
- Database tools: 查询、更新、创建业务记录。

Task decomposition 的本质，就是把业务流程映射到这些 building blocks 的序列。

## Design Takeaways

- Direct generation 是 baseline，不是终点。复杂任务通常需要 workflow。
- 每个步骤最好有清晰的 input/output，这样后续才能 debug、替换组件、写 eval。
- 如果输出不好，先定位是哪一步不好，再考虑是否继续拆解该步骤。
- 不要把“人类直觉上的一个动作”直接当成 agent 的一个步骤；它可能内部包含多个可自动化子步骤。
- Workflow 越复杂，越需要 evals 来驱动迭代，否则只能凭感觉改。

## Failure Modes

- 步骤拆得太粗，导致 LLM 在一个步骤里同时做太多事，错误难以定位。
- 步骤拆得太细，导致 workflow 复杂、成本高、latency 长。
- 每一步没有明确输入输出，后续无法单独评估。
- 只模仿人类流程表面动作，没有思考每一步是否能被现有 tool 实现。
- 初版 workflow 表现不好时，盲目换模型，而不是定位并重构有问题的步骤。

## Implementation Hooks

- Start with baseline: 先做 direct generation 或最简单 workflow，作为比较对象。
- Step inventory: 写出人类完成任务时的步骤，再标注每步由 LLM、tool、API、code 还是 human review 完成。
- Input/output contract: 给每一步定义输入、输出、失败状态和日志。
- Iteration: 对表现差的步骤继续拆解，比如把 "write essay" 拆成 draft、critique、revise。
- Evals: 给关键步骤单独写 eval，再加 end-to-end eval。
- Tool mapping: 每个步骤都明确绑定可用 building block，避免出现“想象中能做、现实没有工具”的步骤。

## My Questions

- 如何判断一个步骤应该继续拆细，还是保持粗粒度？
- workflow 复杂度增加时，如何控制 latency 和 cost？
- 对每个 step 的 eval 应该怎么写，才能帮助定位问题？
- 什么时候应该把某一步交给 human review，而不是继续自动化？

## Related

- [Agentic AI Applications](05-agentic-ai-applications.md)
- [Agentic application fit](../../concepts/agentic-application-fit.md)
- [Task decomposition](../../concepts/task-decomposition.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 task decomposition concept card。
- [x] 下一节 `Evaluating agentic AI (evals)` 后，补充如何为每个 step 设计 eval。
- [ ] 后续 lab 可以用 research agent 做 baseline vs decomposed workflow 对照。
