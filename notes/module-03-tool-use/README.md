# Module 3: Tool Use

## Lesson Checklist

- [x] 01 - What are tools?
- [x] 02 - Creating a tool
- [ ] 03 - Tool syntax
- [ ] 04 - Ungraded Lab: Turning functions into tools
- [ ] 05 - Ungraded Lab: Email Assistant Workflow
- [ ] 06 - Code execution
- [ ] 07 - MCP
- [ ] 08 - Module 3 quiz
- [ ] 09 - M3 Graded Lab

## Module-Level Themes

- Tool use 是让 LLM 可以请求调用开发者提供的 functions。
- Tool use 能让 LLM 获取当前信息、执行动作、查询数据库、计算结果，而不是只靠模型参数里的旧知识。
- 工具调用不等于开发者 hard-code 每一步；关键区别是 LLM 可以根据任务判断是否要调用哪个工具。
- LLM 不直接执行工具；它输出 tool call request，应用层代码执行函数并把结果放回上下文。
- 工具有参数时，tool use 同时包含 tool selection 和 argument filling。
- 多工具场景下，LLM 不仅要生成最终答案，还要选择工具、读取工具结果，再决定下一步。
- 工具越接近外部系统或有副作用，越需要权限、错误处理、日志和 human review。

## Notes

- [01 - What are tools?](01-what-are-tools.md)
- [02 - Creating a tool](02-creating-a-tool.md)
