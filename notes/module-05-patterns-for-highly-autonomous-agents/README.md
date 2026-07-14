# Module 5: Patterns for Highly Autonomous Agents

## Lesson Checklist

- [x] 01 - Planning workflows
- [x] 02 - Creating and executing LLM plans
- [x] 03 - Planning with code execution
- [ ] 04 - Ungraded Lab: Customer Service Agent Code Example
- [ ] 05 - Multi-agentic workflows
- [ ] 06 - Ungraded Lab: Market Research Team Code Example
- [ ] 07 - Communication patterns for multi-agent systems
- [ ] 08 - Module 5 quiz
- [ ] 09 - M5 Graded Assignment: Agentic Workflows
- [ ] 10 - Conclusion
- [ ] 11 - Acknowledgments

## Module-Level Themes

- Highly autonomous agents 不再要求 developer 预先 hard-code 每一步，而是允许 LLM 在 runtime 规划步骤。
- Planning design pattern 的核心是：先生成 plan，再按 plan step-by-step 执行。
- Planning 通常建立在 tool use 之上；LLM 不是只生成答案，而是在多个工具之间决定调用顺序。
- Planner 负责把复杂请求拆成步骤，executor 负责把每个 step 变成工具调用、代码执行或子任务。
- 可执行 plan 最好用 JSON 或 XML 等结构化格式表达，让 downstream code 能稳定解析 step、tool 和 arguments。
- Plan schema 是 planning workflow 和 executor 之间的 API contract；它需要 validation 和 tool allowlist。
- 如果任务天然可以用代码表达，code execution 可以让 LLM 把 plan 直接写成可执行程序。
- Code-as-plan 尤其适合数据表、CSV、pandas、文件处理和计算类任务，但必须配套 sandbox。
- Planning 增强了灵活性，但也降低了可控性：developer 不一定提前知道 runtime 会生成什么 plan。
- Highly agentic coding systems 已经比较成功地使用 planning；其他领域仍在探索如何稳定落地。
- 后续 multi-agent workflow 会继续提升 autonomy，但也会引入更高的协调、调试和评估成本。

## Notes

- [01 - Planning workflows](01-planning-workflows.md)
- [02 - Creating and executing LLM plans](02-creating-and-executing-llm-plans.md)
- [03 - Planning with code execution](03-planning-with-code-execution.md)

## Cross-Cutting Notes

- [Planning pattern](../../concepts/planning-pattern.md): runtime plan generation, step execution, tool sequencing, control tradeoffs.
- [Agentic design patterns](../../concepts/agentic-design-patterns.md): planning 和 reflection、tool use、multi-agent 的关系。
- [Autonomy spectrum](../../concepts/autonomy-spectrum.md): planning 把系统推向更高 autonomy。
- [Task decomposition](../../concepts/task-decomposition.md): planning 可以看成 runtime task decomposition。
- [Tool use](../../concepts/tool-use.md): planning 通常通过工具调用落地。
- [Code execution tool](../../concepts/code-execution-tool.md): code-as-plan、sandbox、execution feedback。
- [Open questions](../../review/open-questions.md): plan quality、runtime control、planning eval。
