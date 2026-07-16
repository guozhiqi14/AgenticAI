# Module 5 Summary

Status: in progress

## Working Thesis

Module 5 的核心是更高 autonomy 的 workflow pattern：不再只让 LLM 在固定步骤中生成内容或调用工具，而是让它在 runtime 规划步骤、编排工具、逐步执行任务。这个能力让 agent 覆盖更宽的请求空间，但也把系统带入更难控制、更难评估的区域。

目前前四个视频主线给出的关键框架是：

- planner 先根据用户请求和可用工具生成 step-by-step plan。
- executor 再按计划逐步运行，每一步把前一步结果带入下一步。
- plan 是可执行的中间 artifact，需要被 logging、guardrail 和 eval。
- plan 最好用 JSON 或 XML 等结构化格式表达，这样 downstream code 才能 parse、validate 和 execute。
- 当任务适合写代码时，LLM 可以把 plan 直接表达成 Python/pandas 等可执行代码。
- 当任务天然对应多个专业角色时，可以把 workflow 拆成多个 agents 协作完成。

## Lessons Covered

- [x] Planning workflows
- [x] Creating and executing LLM plans
- [x] Planning with code execution
- [ ] Ungraded Lab: Customer Service Agent Code Example
- [x] Multi-agentic workflows
- [ ] Ungraded Lab: Market Research Team Code Example
- [ ] Communication patterns for multi-agent systems
- [ ] Module 5 quiz
- [ ] M5 Graded Assignment: Agentic Workflows
- [ ] Conclusion
- [ ] Acknowledgments

## Ideas To Revisit

- Planning 是 runtime task decomposition：模型为当前请求决定步骤。
- Planning 通常建立在 tool use 之上；tools 提供能力，planner 决定调用顺序。
- Sunglasses customer service 例子说明：同一组工具可以回答库存/价格问题，也可以处理退货问题。
- Email assistant 例子说明：后一步依赖前一步结果，必须先找到目标邮件，才能回复和归档。
- Coding agent 里的 checklist/build-plan 是 planning pattern 的典型成功场景。
- Planning 的主要收益是灵活性；主要代价是可控性下降。
- Developer 不一定提前知道 runtime 会生成什么 plan，因此必须记录 plan 和 step-level traces。
- Plan schema 是 planner 和 executor 之间的 contract；字段设计会影响 parsing、validation、trace 和 eval。
- JSON/XML 比 markdown/plain text 更适合作为可执行 plan，因为边界更明确、解析更稳定。
- Invalid JSON、未知 tool name、错误 arguments 和过宽松 schema 都会让 executor 不稳定。
- Code-as-plan 可以减少 tool explosion：不用为每个数据操作都封装一个新工具。
- Python/pandas 适合表达数据分析 plan，因为它天然支持过滤、聚合、排序、去重、日期处理和中间变量。
- Code execution 让 plan 更有表达力，但 sandbox、timeout、resource limit、file/network policy 是硬边界。
- Planning 在 agentic coding 中最成熟；其他业务领域仍然需要 guardrails 和 eval 才能放心落地。
- Multi-agent workflow 的本质是 role decomposition：researcher、designer、writer、editor、manager 等角色各自承担不同子任务。
- 不同 agent 可以拥有不同 tools；例如 researcher 需要 search，designer 需要 image/chart tools，writer 可能只需要 LLM writing。
- 线性 multi-agent workflow 容易实现和 debug；manager-agent workflow 更灵活，manager 可以把 specialist agents 当作可调用对象来委派任务。
- Multi-agent 的核心架构问题是 communication pattern：谁和谁通信、传什么上下文、谁负责最终整合和 review。
- Planning 不应该默认用于所有流程；稳定、高风险、合规强的流程可能更适合 deterministic workflow。

## Self-Test Questions

- Planning pattern 和 hard-coded workflow 的区别是什么？
- 为什么 planning 可以覆盖更宽的用户请求空间？
- Planner 和 executor 分别承担什么职责？
- 为什么 plan 应该被看作一个可观察的中间 artifact？
- Sunglasses customer service 例子里，planner 为什么需要先查商品描述再查库存和价格？
- Email assistant 例子里，为什么工具调用顺序不能随便调换？
- Planning 为什么在 coding agent 里比较成功？
- Planning workflow 为什么更难控制和 debug？
- Plan quality 可以从哪些角度评估？
- 为什么可执行 plan 更适合用 JSON 或 XML，而不是 plain text？
- Plan schema 里至少应该包含哪些字段？
- Parser/validator 在 planner 和 executor 之间起什么作用？
- Invalid JSON 或未知 tool name 应该如何处理？
- 为什么一堆细粒度数据处理工具会导致 tool explosion？
- Code-as-plan 相比 JSON plan 的优势和风险分别是什么？
- 为什么 code execution 必须配 sandbox，而不是只靠 prompt 约束？
- 对 CSV/pandas 分析 agent，如何判断代码逻辑真的正确？
- 为什么 multi-agent 不只是“多调用几次 LLM”？
- 线性 multi-agent workflow 和 manager-agent workflow 的区别是什么？
- Specialist agent 的 role boundary 应该如何定义？
- 为什么不同 agent 应该有不同 tool permissions？
- Multi-agent final output 错误时，如何定位是哪一个 agent 或通信环节出了问题？
- 对数据分析 agent，哪些任务适合 planning，哪些应该保持 fixed workflow？
