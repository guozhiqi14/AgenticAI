# Module 2 Summary

Status: in progress

## Working Thesis

Reflection 是一种低门槛但实用的 agentic design pattern。它通过 draft、critique/feedback、revise 的循环提升输出质量；当 reflection 能接入外部反馈时，比如错误日志、测试结果或用户反馈，它通常比纯自我反思更有效。

## Lessons Covered

- [x] Reflection to improve outputs of a task
- [x] Why not just direct generation?

## Ideas To Revisit

- Reflection 不是魔法，通常带来 modest performance bump。
- External feedback 会显著增强 reflection。
- Reasoning model 可能更适合 critique/debug。
- Reflection 的收益需要通过 eval 验证，而不是凭感觉判断。
- Direct generation / zero-shot prompting 是 reflection 的 baseline。
- Reflection prompt 要明确 review/reflect，并给出具体 criteria。
- Reflection 更适合格式、完整性、事实、语气、负面含义等可检查维度明确的任务。

## Self-Test Questions

- Reflection pattern 的基本流程是什么？
- 为什么 external feedback 会让 reflection 更强？
- 什么类型的任务适合用 reflection？
- Reflection 有哪些 failure modes？
- direct generation、zero-shot、one-shot、few-shot 的区别是什么？
- 为什么 criteria 具体化会提升 reflection prompt 的效果？
- 如何判断 reflection 相比 direct generation 是否值得？
