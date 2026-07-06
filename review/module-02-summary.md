# Module 2 Summary

Status: in progress

## Working Thesis

Reflection 是一种低门槛但实用的 agentic design pattern。它通过 draft、critique/feedback、revise 的循环提升输出质量；当 reflection 能接入外部反馈时，比如错误日志、测试结果或用户反馈，它通常比纯自我反思更有效。

但 reflection 不应该默认保留。它会增加 latency 和 cost，所以需要通过 eval 比较 no-reflection baseline 和 reflection workflow 的质量提升是否值得。

这个 module 的关键判断是：先用 direct generation 建 baseline；如果质量不够，再尝试 reflection；如果 prompt tuning 或 self-reflection 进入平台期，再寻找 external feedback，比如代码执行、检索、规则检查或用户反馈。

## Lessons Covered

- [x] Reflection to improve outputs of a task
- [x] Why not just direct generation?
- [x] Evaluating the impact of reflection
- [x] Using external feedback

## Ideas To Revisit

- Reflection 不是魔法，通常带来 modest performance bump。
- External feedback 会显著增强 reflection。
- Reasoning model 可能更适合 critique/debug。
- Reflection 的收益需要通过 eval 验证，而不是凭感觉判断。
- Direct generation / zero-shot prompting 是 reflection 的 baseline。
- Reflection prompt 要明确 review/reflect，并给出具体 criteria。
- Reflection 更适合格式、完整性、事实、语气、负面含义等可检查维度明确的任务。
- SQL/database query 这类任务适合 objective eval，因为可以和 ground truth answer 对比。
- 图表、报告质量等主观任务可以用 LLM-as-judge，但最好用 binary rubric，而不是让模型直接做 A/B 偏好选择。
- LLM-as-judge 可能有 position bias；pairwise comparison 的结果不一定稳定。
- External feedback 的价值在于提供新信息，而不是让模型在同一份上下文里继续自我确认。
- 简单工具也能提供强反馈，比如 regex 检查竞品名、word count 检查字数、代码执行返回错误日志。
- External feedback 是 reflection 到 tool use 的过渡：工具调用会让 agentic workflow 更系统地获得外部信息。

## Self-Test Questions

- Reflection pattern 的基本流程是什么？
- 为什么 external feedback 会让 reflection 更强？
- 什么类型的任务适合用 reflection？
- Reflection 有哪些 failure modes？
- direct generation、zero-shot、one-shot、few-shot 的区别是什么？
- 为什么 criteria 具体化会提升 reflection prompt 的效果？
- 如何判断 reflection 相比 direct generation 是否值得？
- 为什么要先建立 no-reflection baseline？
- Objective eval 和 rubric-based LLM-as-judge 分别适合什么任务？
- 为什么直接让 LLM 比较两张图哪张更好不够可靠？
- Binary rubric 为什么通常比 1-5 分更容易校准？
- 为什么 external feedback 通常比 pure self-reflection 更强？
- 举三个可以作为 external feedback 的简单工具。
- 当 prompt tuning 进入 plateau 时，为什么应该考虑工具反馈，而不是继续只改 prompt？
