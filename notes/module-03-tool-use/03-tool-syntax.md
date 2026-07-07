---
course: Agentic AI
module: 3
lesson: 3
title: Tool Syntax
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/154qpw/tool-syntax
watched_at: 2026-07-07
reviewed_at:
tags: [tool-use, tool-schema, json-schema, aisuite, function-calling, max-turns]
---

# Tool Syntax

## TL;DR

Tool syntax 是把普通函数变成 LLM 可理解工具的“接口说明”。现代 tool-calling API 通常需要把函数名、函数描述、参数 schema 和可用工具列表传给模型。模型不会自己执行函数；它根据这些说明生成 tool call request，外层应用或 library 负责执行函数、把结果放回上下文，并让模型继续下一步。

## Core Ideas

- 开发者需要把可用 tools 传给 LLM，而不是只在 prompt 里口头说“你可以调用函数”。
- Tool schema 会告诉模型：工具叫什么、做什么、需要哪些参数、参数是什么类型或格式。
- `AISuite` 这类 library 可以自动从 Python function、docstring 和参数说明生成 JSON schema。
- 有些 API 需要开发者手动构造 JSON schema；有些 library 会把这一步封装起来。
- 工具调用可能连续发生：模型拿到一个 tool result 后，可能还想调用下一个工具。
- `max_turns` 是一个保护上限，防止模型连续请求工具调用时陷入无限循环。
- Code execution 是一种特别强的工具，因为 LLM 可以写代码，再让外部系统执行代码。

## Mental Model

把 tool schema 想成你给模型看的函数使用说明书：

```text
function name: get_current_time
description: returns the current time for a given timezone
parameters:
  timezone:
    type: string
    example: America/New_York, Pacific/Auckland
```

模型看到这份说明书后，才更容易判断：

- 什么时候应该调用这个工具。
- 应该传什么参数。
- 工具返回结果应该如何用于最终回答。

## Example: No-Argument Tool

如果工具不需要参数，比如：

```python
def get_current_time():
    """Return the current time."""
    ...
```

系统可以把函数名和 docstring 转成工具描述。用户问“现在几点？”时，模型可以请求调用 `get_current_time`。外层应用执行函数后，把结果返回给模型，模型再回答用户。

这里的重点是：函数本身仍然是普通 Python 代码；tool syntax 只是让 LLM 知道这个函数存在，以及什么时候适合请求调用。

## Example: Tool With Arguments

带参数的工具更接近真实场景：

```python
def get_current_time(timezone: str):
    """Return the current time in the specified timezone."""
    ...
```

这时 schema 不只要描述函数本身，还要描述参数：

- 参数名：`timezone`
- 参数类型：`string`
- 参数含义：要查询的时区
- 参数示例：`America/New_York`、`Pacific/Auckland`

这会帮助模型把“New Zealand 现在几点？”转成类似 `timezone = "Pacific/Auckland"` 的 tool arguments。

## AISuite Flow

课程里用 `AISuite` 展示了一种封装后的调用方式。大致流程是：

```text
messages + model + tools + max_turns
  -> LLM decides whether to request a tool call
  -> AISuite executes the requested function
  -> AISuite feeds tool result back to the LLM
  -> LLM returns answer or requests another tool call
```

也就是说，开发者不用在这一层手写每一次 `parse -> execute -> append tool result -> call LLM again`，library 可以把这个 loop 包起来。

但底层机制没有变：模型仍然只是请求工具调用，真正执行函数的是外部系统。

## JSON Schema

Tool schema 通常会以 JSON schema 形式传给模型。它像一份结构化 contract：

```text
tool:
  name
  description
  parameters:
    type
    properties
    required fields
```

对数据分析 agent 来说，这一步很关键。比如 SQL 工具如果只写成 `query_database(sql)`，模型可能乱写 SQL。更稳的方式可能是补充清楚：

- 可以查询哪些表。
- 参数是 raw SQL 还是结构化 query plan。
- 是否允许写操作。
- 最大返回行数是多少。
- 错误时返回什么格式。

## Max Turns

`max_turns` 是 tool-call loop 的安全阀。

```text
LLM requests tool A
  -> tool A result
  -> LLM requests tool B
  -> tool B result
  -> ...
```

如果不设上限，模型或 workflow bug 可能导致连续工具调用停不下来。实际简单任务通常不会碰到这个限制，但设置一个合理上限能让系统更安全、成本更可控。

## Design Takeaways

- Tool syntax 的目标不是写“更花的 prompt”，而是给 LLM 明确、结构化的工具接口。
- Docstring 不只是给人看的注释；在某些 library 里，它会直接变成 LLM 看到的 tool description。
- 参数说明越清楚，模型越容易生成正确 tool arguments。
- 对有副作用或高成本工具，schema 里要尽量收窄参数空间，并在执行层做 validation。
- Library 自动执行 tool-call loop 很方便，但你仍然要理解背后的 request -> execute -> return 过程。
- `max_turns`、logging 和 error handling 是工具系统进入真实应用前必须考虑的工程保护。

## Failure Modes

- Docstring 太泛，模型不知道什么时候该调用工具。
- 参数没有类型、格式或示例，导致模型传错 arguments。
- 自动生成 schema 后不检查，结果把模糊或危险的接口暴露给模型。
- 把 library 封装当成魔法，忘记 tool result 必须回到上下文。
- `max_turns` 设得太高或没有停止条件，导致循环调用、成本上升。
- 对 code execution 这类强工具缺少沙箱、权限和资源限制。

## Implementation Hooks

- 写工具函数时，先写清楚 docstring：它做什么、什么时候用、参数含义是什么。
- 为参数加类型提示和示例，尤其是 timezone、date range、table name、metric name 这类容易歧义的字段。
- 自动生成 tool schema 后，打印出来人工检查一次。
- 对 tool arguments 做 validation，再执行真实函数。
- 保存每次 tool schema、tool call request、arguments、result、error 和 turn count。
- 对连续工具调用设置 `max_turns`，并在达到上限时返回可诊断错误。

## My Questions

- 数据分析 agent 的 SQL 工具应该暴露 raw SQL，还是更结构化的 query spec？
- 我们能不能把表 schema、指标口径、权限范围也纳入 tool schema 或 tool description？
- Docstring 自动生成 schema 很方便，但如何保证描述足够准确、不会误导模型？
- 对 code execution tool，最小安全沙箱应该包含哪些限制？
- Tool schema 改动后，应该怎么做 regression eval，确认 tool selection 和 arguments 没变差？

## Related

- [What are tools?](01-what-are-tools.md)
- [Creating a tool](02-creating-a-tool.md)
- [Tool use](../../concepts/tool-use.md)
- [Tool schema](../../concepts/tool-schema.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 3 README。
- [x] 新增 tool schema concept card。
- [x] 补充 glossary 中的 tool schema、JSON schema、docstring、max turns。
- [ ] 下一节 lab 后，记录 function -> tool 的可运行最小例子。
- [ ] 后续 code execution 视频后，补充 code execution tool 的安全边界。
