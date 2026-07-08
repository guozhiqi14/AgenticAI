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

## Tool Schema

现代 tool use 通常会把工具以结构化 schema 暴露给 LLM。这个 schema 像一份“给模型看的 API 文档”：

```text
tool name
tool description
parameters
required fields
argument types / examples
```

有些 library 可以从 Python function name、docstring、type hints 和参数说明自动生成 JSON schema。这样很方便，但也让函数注释变得更重要：docstring 写得含糊，模型看到的 tool description 也会含糊。

对于数据分析 agent，tool schema 应该特别说清楚表名、指标名、日期范围、返回行数、是否允许写操作、错误返回格式等边界。

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

## Code Execution As A Special Tool

Code execution tool 不是一个固定业务函数，而是让 LLM 为当前任务写代码，再由外部执行环境运行代码。它适合计算、数据处理、绘图、文件解析和 sanity check。

这个工具很强，因为一个 code execution tool 可以覆盖很多临时能力；但它也更危险，因为 LLM-generated code 可能读写文件、访问网络、占用资源或产生副作用。因此，生产系统里应该把 code execution 放在 sandbox 中，并限制权限、资源、网络和执行时间。

## MCP As Tool Ecosystem

MCP 把 tool use 从“每个应用自己封装每个工具”推进到标准化生态。应用可以作为 MCP client，连接 GitHub、Postgres、Slack、Google Drive 或内部业务系统的 MCP server，获取 resources 或调用 tools。

这对 agent 应用很重要，因为它减少重复集成工作，也让工具/数据源更容易被多个 LLM 应用复用。

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
- 是否已有 MCP server 可以复用，而不是自己重新封装 API？
- 工具是否应该由 LLM 自主选择，还是由 workflow 固定调用？
- 工具输入参数是否足够清楚？
- Tool schema 是否准确描述了 name、description、parameters 和 required fields？
- 自动生成的 schema 是否经过人工检查？
- 工具返回结果是否容易被 LLM 使用？
- 工具失败时应该返回什么？
- 工具是否有副作用？是否需要 human approval？
- 如果工具是 code execution，是否有 sandbox、timeout 和资源限制？
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
- Docstring 或自动生成 schema 太模糊，导致模型误用工具。
- 工具返回过多原始数据，超过上下文或难以总结。
- LLM 过度调用工具，造成成本和延迟上升。
- LLM 不调用必要工具，生成 hallucinated answer。
- 有副作用工具缺少权限控制，造成错误操作。
- Code execution 没有隔离，导致文件损坏、数据泄露或资源耗尽。

## Related Notes

- [What are tools?](../notes/module-03-tool-use/01-what-are-tools.md)
- [Creating a tool](../notes/module-03-tool-use/02-creating-a-tool.md)
- [Tool Syntax](../notes/module-03-tool-use/03-tool-syntax.md)
- [Code Execution](../notes/module-03-tool-use/06-code-execution.md)
- [MCP](../notes/module-03-tool-use/07-mcp.md)
- [Tool schema](tool-schema.md)
- [Code execution tool](code-execution-tool.md)
- [Model Context Protocol](model-context-protocol.md)
- [External feedback](external-feedback.md)
- [Task decomposition](task-decomposition.md)
- [Agentic design patterns](agentic-design-patterns.md)
