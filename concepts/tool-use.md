# Concept: Tool Use

## Definition

Tool use 是让 LLM 可以请求调用开发者提供的函数、API、数据库查询、搜索、代码执行或其他外部能力。工具本身在 LLM 外部执行，执行结果再返回给 LLM，用来继续推理或生成最终答案。

## Why It Matters

LLM 不能只靠模型参数解决所有问题。它不知道实时状态，不能直接访问私有数据库，也不擅长精确计算。Tool use 把 LLM 的语言理解和决策能力，连接到真实系统、实时信息和确定性计算。

## Basic Shape

```text
User request
  -> LLM decides whether a tool is needed
  -> LLM requests tool call with arguments
  -> Tool runs outside the LLM
  -> Tool result returns to LLM
  -> LLM answers or calls another tool
```

## Tool Call Loop

Tool use 的关键不是 LLM 自己执行函数，而是应用层帮它执行：

```text
LLM emits tool call request
  -> system parses tool name and arguments
  -> system calls real function/API
  -> system returns tool result to LLM
  -> LLM continues
```

早期可以用文本约定实现，比如让模型输出 `FUNCTION: getCurrentTime`；现代 LLM 通常有更标准的结构化 tool-call syntax。

## Tool Use vs Hard-Coded Tool Call

| Pattern | Decision Maker | Strength | Risk |
| --- | --- | --- | --- |
| Hard-coded tool call | Developer | 可控、稳定、容易 eval | 不够灵活 |
| LLM-selected tool use | LLM | 灵活、能适配不同请求 | 可能选错工具、传错参数、漏用工具 |

## Common Tool Types

| Tool Type | Example |
| --- | --- |
| Current state | current time, weather, calendar availability |
| Search/retrieval | web search, business wiki, document search |
| Database | SQL query, customer/order lookup |
| Calculation | interest calculator, Python code execution |
| Business action | send email, create calendar invite, update CRM |
| Validation | schema check, word count, policy checker |

## Tool Arguments

有些工具不需要参数，例如 `getCurrentTime()`。更多工具需要 arguments，例如 `getCurrentTime(timezone)`、`queryDatabase(sql)`、`searchWeb(query)`。

Tool use 的稳定性很大程度取决于 argument design：

- 参数名是否清楚。
- 参数类型是否明确。
- 是否有必填/可选字段。
- 是否限制枚举值或安全范围。
- 参数错了是否能返回清楚错误。

## Design Questions

- 用户任务需要哪些外部信息或动作？
- 工具是否应该由 LLM 自主选择，还是由 workflow 固定调用？
- 工具输入参数是否足够清楚？
- 工具返回结果是否容易被 LLM 使用？
- 工具失败时应该返回什么？
- 工具是否有副作用？是否需要 human approval？
- 如何评估 tool selection 和 tool arguments 是否正确？
- Tool result 如何放回上下文，才能让 LLM 正确继续？

## Practical Rule

先从任务中最稳定、最有价值的外部能力做工具。对数据分析 agent 来说，常见优先级可能是：

1. schema/table metadata lookup
2. SQL query execution
3. metric definition retrieval
4. Python calculation / plotting
5. report or chart validation

不要一开始给 LLM 太多工具。工具越多，选择空间越大，debug 和 eval 难度也会上升。

## Failure Modes

- Tool description 太模糊，LLM 不知道何时调用。
- 参数 schema 不清楚，导致 tool arguments 错误。
- 工具返回过多原始数据，超过上下文或难以总结。
- LLM 过度调用工具，造成成本和延迟上升。
- LLM 不调用必要工具，生成 hallucinated answer。
- 有副作用工具缺少权限控制，造成错误操作。

## Related Notes

- [What are tools?](../notes/module-03-tool-use/01-what-are-tools.md)
- [Creating a tool](../notes/module-03-tool-use/02-creating-a-tool.md)
- [External feedback](external-feedback.md)
- [Task decomposition](task-decomposition.md)
- [Agentic design patterns](agentic-design-patterns.md)
