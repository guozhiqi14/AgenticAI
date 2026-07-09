# Open Questions

## Module 1

- 实现 agentic workflow 前，如何决定合适的 autonomy level？
- 哪些 eval 能评估 action selection，而不只是 final output quality？
- tool choice 什么时候应该 hard-coded，什么时候应该 model-selected，什么时候应该 human-approved？
- 什么 stopping condition 能让 reflection loop 既安全又成本可控？
- 为了 debug，agent workflow 的哪些部分必须 logging？
- 什么时候 agentic workflow 的性能提升足以抵消额外 latency 和 cost？
- 如何评价替换某个 tool、search engine 或 LLM provider 后是否真的变好？
- 如何判断一个业务流程是否已经足够 SOP 化，适合先做 agentic workflow？
- 什么时候应该接受 browser/computer use 的不稳定性，而不是要求稳定 API？
- human review 应该放在最终输出前，还是放在每个高风险 tool call 前？
- 如何判断一个 workflow step 应该继续拆细，还是保持当前粒度？
- task decomposition 变复杂后，如何平衡质量提升、latency 和 cost？
- 如何把每个 discrete step 的输入输出设计得足够稳定，方便 eval 和调试？
- 我自己的 agent 项目里，哪些真实 failure patterns 应该优先转成 eval？
- LLM-as-judge 的 rubric 如何设计，才能比简单 1-5 分更可靠？
- 如何避免 eval 指标提升但真实用户体验没有提升？
- 在实际项目里，应该先用 reflection、tool use、planning 还是 multi-agent？
- planning 的 plan quality 应该怎么评估？
- multi-agent workflow 什么时候真的比单 agent workflow 更好？
- reflection 需要哪些外部反馈，才能避免模型只是自我确认？
- 哪些任务最适合用 reflection，哪些任务直接 generation 就够了？
- Critique prompt 应该如何写，才能让 feedback 具体可执行？
- Reflection loop 的停止条件应该如何设计？
- Reflection prompt 中 criteria 应该写到多细，才能既有效又不限制模型发现其他问题？
- 如何设计 direct generation vs reflection 的最小对照实验？
- 如何区分 reflection 带来的“表面润色”与“真实质量提升”？

## Module 2

- 对我自己的数据分析 agent，哪些任务可以最先建立 prompts + ground truth 的 objective eval set？
- 对图表和分析报告，binary rubric 应该怎么设计，才能既贴近业务质量，又不让 LLM-as-judge 过度主观？
- 当 reflection workflow 的 eval 分数提升但 latency/cost 也上升时，应该如何设定“值得上线”的阈值？
- 对数据分析 agent，哪些 external feedback 最应该先做：SQL execution error、schema validation、指标口径校验、还是业务 wiki retrieval？
- 如何设计 feedback format，才能让 LLM 根据工具结果稳定 revision，而不是忽略或误解反馈？
- 哪些 external feedback 可以自动触发 revision，哪些应该触发 human review？

## Module 3

- 数据分析 agent 的第一批 tools 应该如何排序：schema lookup、SQL execution、metric retrieval、Python execution、chart generation？
- 如何评估 LLM 是否在正确时机调用正确工具？
- Tool result 应该返回多细：原始 rows、聚合结果、摘要、还是结构化 JSON？
- 哪些工具可以让 LLM 自主调用，哪些工具必须 human-approved？
- Tool 数量增加后，如何避免模型选择困难或误调用？
- Tool arguments 应该让 LLM 直接生成自由文本，还是尽量设计成结构化字段和枚举值？
- Tool result 很长时，应该如何摘要或采样，避免把无关信息塞回上下文？
- 数据分析 agent 的 SQL 工具 schema 应该暴露 raw SQL，还是暴露更受控的 query spec？
- Tool schema 里应该如何表达表权限、指标口径、返回行数上限和禁止写操作？
- 自动从 docstring 生成 tool schema 后，应该如何 review 和 eval，避免描述误导模型？
- `max_turns` 到底应该按任务类型固定，还是根据工具风险动态设置？
- Code execution tool 的最小安全沙箱应该包含哪些限制？
- 数据分析 agent 的 Python execution tool 是否应该默认禁止网络访问？
- Code execution 生成的 artifact 应该如何保存、命名和清理？
- 如何评估 LLM 写代码的质量：看最终答案、代码可读性、执行成功率，还是安全违规率？
- 当 code execution 失败时，哪些错误适合自动 retry，哪些应该直接交给人？
- 数据分析 agent 最应该先接哪个 MCP server：GitHub、Postgres、业务 wiki、指标平台，还是实验平台？
- MCP resource 返回原始内容、摘要还是结构化 JSON，哪个更利于稳定回答？
- 多个 MCP servers 同时可用时，如何评估 LLM 是否选对 server/tool/resource？
- 内部 MCP server 的权限应该如何和员工身份、数据表权限、业务动作权限对齐？

## Module 4

- 数据分析 agent 的第一批 10 到 20 个 eval examples 应该从哪些真实任务抽样？
- 哪些失败模式最适合先做 objective eval：SQL 执行失败、指标口径错误、字段缺失、图表格式，还是结论缺 evidence？
- 对分析报告和图表，LLM-as-a-judge 的 rubric 应该拆成哪些 binary criteria？
- 如果 eval metric 和我的人工判断冲突，应该优先调整 eval set、judge prompt，还是 workflow？
- Eval set 多久扩充一次，才能避免系统只对旧样例过拟合？
- End-to-end eval 分数下降时，如何快速定位应该转向哪个 component-level eval？
- 数据分析 agent 的 trace/span 记录字段应该如何设计，才能既方便 debug 又不泄露敏感数据？
- 当上游 schema lookup 错误导致 SQL 错误时，error analysis 应该如何归因？
- Component 错误频率和修复成本如何量化，才能更客观地排优先级？
- 数据分析 agent 的 error tags 应该允许多选到什么粒度，才能既真实又不难维护？
- SQL generation 错误和 database/data quality 错误应该如何拆开评估？
- 如果 final answer 写得不好，但上游 SQL/result 也有问题，component attribution 应该如何避免重复计数？
- 数据分析 agent 应该先做哪个 component-level eval：schema retrieval、SQL generation、SQL execution validation，还是 chart spec generation？
- Component-level eval 的局部指标要多高，才值得跑一次更贵的 end-to-end eval？
- 如何避免 component-level metric 变好，但 end-to-end user experience 没有变好？
