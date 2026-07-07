# Concept: Tool Schema

## Definition

Tool schema 是给 LLM 看的工具接口说明，通常包含 tool name、description、parameters、required fields 和参数类型。它帮助模型判断什么时候调用工具，以及如何生成正确的 tool arguments。

可以把它理解成“给模型看的 API 文档”，但它要比自然语言文档更结构化。

## Why It Matters

LLM 不是靠猜来稳定使用工具的。它需要清楚知道：

- 工具有哪些。
- 每个工具做什么。
- 每个工具需要哪些参数。
- 参数应该是什么类型、格式和范围。
- 工具失败时可能返回什么。

Tool schema 写得越清楚，tool selection 和 argument filling 越稳定。

## Basic Shape

```text
tool:
  name: get_current_time
  description: Return the current time for a timezone.
  parameters:
    timezone:
      type: string
      description: IANA timezone, such as Pacific/Auckland.
  required:
    - timezone
```

现代 tool-calling API 往往用 JSON schema 或类似结构表达这份说明。

## Docstring To Schema

有些 library 可以从 Python function 自动生成 tool schema：

```python
def get_current_time(timezone: str):
    """Return the current time in the specified timezone."""
    ...
```

自动生成通常会利用：

- function name -> tool name
- docstring -> tool description
- type hints / parameter docs -> parameter schema

这很方便，但也意味着 docstring 会影响模型行为。含糊的 docstring 会变成含糊的工具说明。

## Design Questions

- Tool description 是否说明了“什么时候应该用”？
- 参数是否有清楚类型、格式、示例和限制？
- 哪些参数应该 required，哪些可以 optional？
- 是否应该用 enum 限制取值？
- 是否需要把权限、成本、副作用写进 schema 或执行层 validation？
- 自动生成 schema 后，是否需要人工 review？
- Tool result 的 schema 是否也需要固定？

## Practical Rule

对高频、高价值工具，不要只依赖“函数名看起来懂”。至少写清楚三件事：

1. 这个工具解决什么问题。
2. 什么时候不应该调用它。
3. 每个参数的合法格式和例子。

对数据分析 agent，尤其要小心 table name、metric name、date range、aggregation level、row limit、write permission 这些字段。

## Failure Modes

- 工具描述太泛，模型过度调用或漏调用。
- 参数说明不清，模型传入自然语言而不是合法结构。
- 自动 schema 暴露了不该让模型自由控制的参数。
- Tool result 没有固定结构，模型难以可靠读取。
- Schema 改动后没有 eval，导致 tool calling 行为悄悄退化。

## Related Notes

- [Tool Syntax](../notes/module-03-tool-use/03-tool-syntax.md)
- [Creating a tool](../notes/module-03-tool-use/02-creating-a-tool.md)
- [Tool use](tool-use.md)
- [Agentic evals](agentic-evals.md)
