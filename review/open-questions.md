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
