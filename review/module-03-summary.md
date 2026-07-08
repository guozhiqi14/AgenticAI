# Module 3 Summary

Status: videos and notes covered; quiz and graded lab pending

## Working Thesis

Tool use 把 LLM 从“只会生成文本”扩展成“可以请求外部函数获取信息或执行动作”的系统。核心设计问题不只是写函数，而是决定：给模型哪些工具、工具怎么描述、参数如何约束、什么时候需要人工确认、如何评估 tool selection 是否正确。

## Lessons Covered

- [x] What are tools?
- [x] Creating a tool
- [x] Tool syntax
- [x] Ungraded Lab: Turning functions into tools
- [x] Ungraded Lab: Email Assistant Workflow
- [x] Code execution
- [x] MCP

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
- Code execution 是特殊工具：LLM 写代码，系统执行代码，再把 output/error/artifacts 放回上下文。
- Code execution 和 reflection 可以结合：代码失败时，把 traceback 作为 external feedback 让模型修正。
- 真实系统不要只靠 prompt 约束 LLM-generated code，应该用 sandbox、timeout、resource limit 和权限控制。
- MCP 是标准化接入 tools/resources 的协议，核心角色是 MCP client 和 MCP server。
- MCP 的价值是复用工具集成，把重复 wrapper 工作从 `M * N` 降到接近 `M + N`。
- MCP resources 更偏读取上下文，MCP tools 更偏调用动作；两者都需要权限、trace 和 eval。
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
- 为什么 code execution 比固定数学工具更灵活？
- Code execution tool 的基本 loop 是什么？
- 为什么 traceback 是一种 external feedback？
- Sandbox 需要限制哪些东西？
- MCP 试图解决什么集成问题？
- MCP client 和 MCP server 分别是什么？
- Resource 和 tool 在 MCP 里有什么区别？
- 为什么 MCP 不能替代权限控制和 eval？
- 为什么 current time、web search、database query、calculator 都适合作为工具？
- 多工具 calendar assistant 为什么需要先 `check_calendar` 再 `make_appointment`？
- 有副作用的工具为什么需要更严格的权限控制？
