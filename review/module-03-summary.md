# Module 3 Summary

Status: in progress

## Working Thesis

Tool use 把 LLM 从“只会生成文本”扩展成“可以请求外部函数获取信息或执行动作”的系统。核心设计问题不只是写函数，而是决定：给模型哪些工具、工具怎么描述、参数如何约束、什么时候需要人工确认、如何评估 tool selection 是否正确。

## Lessons Covered

- [x] What are tools?
- [x] Creating a tool
- [x] Tool syntax

## Ideas To Revisit

- Tools 本质上是开发者提供的 functions。
- LLM 可以根据任务判断是否调用工具；这不同于 developer hard-code 每一步工具调用。
- LLM 不直接执行函数，而是输出 tool call request；应用层代码负责解析、执行、返回结果。
- Tool arguments 是稳定性的关键：选对工具但参数错，最终仍然会错。
- Tool schema 是模型能否稳定调用工具的接口说明，通常包含 name、description、parameters 和 required fields。
- Docstring、type hints 和参数说明可能会被 library 自动转成 JSON schema，所以函数注释会直接影响模型行为。
- 工具返回值会被放回上下文，让 LLM 基于新信息继续回答或调用下一个工具。
- 多工具场景下，LLM 需要选择工具和组合工具，比如 calendar assistant 先查空档再创建日程。
- `max_turns` 是连续工具调用的安全上限，用来避免 tool-call loop 失控。
- Tool use 和 external feedback 是连续的：工具结果经常就是给 LLM 的新证据。
- 有副作用的工具需要权限控制和 human review。

## Self-Test Questions

- Tool use 的基本流程是什么？
- Tool 和普通函数有什么关系？
- LLM-selected tool use 和 hard-coded tool call 有什么区别？
- 为什么说 LLM 只是 request function call，而不是自己执行函数？
- 一个 tool call loop 包含哪些步骤？
- 有参数的工具为什么比无参数工具更容易出错？
- Tool schema 一般包含哪些信息？
- 为什么 docstring 会影响 LLM 的 tool calling 行为？
- 自动生成 JSON schema 有什么好处和风险？
- `max_turns` 解决的是什么问题？
- 为什么 current time、web search、database query、calculator 都适合作为工具？
- 多工具 calendar assistant 为什么需要先 `check_calendar` 再 `make_appointment`？
- 有副作用的工具为什么需要更严格的权限控制？
