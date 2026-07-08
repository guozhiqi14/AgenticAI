# Concept: External Feedback

## Definition

External feedback 是来自 LLM 自身之外的新信息，用来帮助 reflection 或 revision。它可以来自代码执行、错误日志、测试结果、检索结果、业务规则、数据库查询、用户反馈或简单工具检查。

Tool use 可以看作 external feedback 的更系统版本：LLM 不只是被动接收反馈，还可以主动请求调用某个工具来获取信息或执行动作。

## Why It Matters

没有 external feedback 时，reflection 只是让模型基于同一份上下文再思考一次，容易自我确认。加入 external feedback 后，系统获得了新的 evidence 或确定性检查结果，因此更容易发现具体问题并修正。

## Common Sources

| Source | Feedback Example | Useful For |
| --- | --- | --- |
| Code execution | traceback, stdout, stderr | 代码生成、数据处理 |
| Unit tests / evals | failed assertion, expected vs actual | 行为正确性 |
| Pattern matching | 命中禁用词、竞品名、敏感词 | 文案和合规检查 |
| Retrieval / search | trusted source snippet | 事实修正 |
| Deterministic checks | word count, JSON schema, SQL execution | 格式、长度、结构、查询正确性 |
| User feedback | 人指出的错误或偏好 | 产品体验和主观质量 |

## Basic Shape

```text
Input
  -> Generate draft
  -> Run checker/tool/search/test
  -> Feed external feedback to LLM
  -> Revise
  -> Final output
```

## Design Questions

- 这个任务有没有可执行、可检索、可验证的外部信号？
- 这个 feedback 是否足够具体，能指导模型修改？
- Feedback source 是否可信？
- 工具返回的是原始结果，还是结构化 issue list？
- 哪些问题可以自动 revise，哪些需要 human review？
- External feedback 带来的质量提升是否值得 latency 和 cost？

## Practical Rule

当 prompt tuning 开始 plateau 时，不要只继续改 prompt。先问：有没有一个小工具可以稳定告诉模型哪里错？

小工具也有大价值。一个 regex checker、word counter、SQL executor 或 schema validator，可能比更复杂的 critique prompt 更可靠。

## Failure Modes

- Feedback 太泛，只告诉模型“质量不好”，没有证据。
- Feedback source 不可信，导致模型被错误信息带偏。
- 工具只检查表面指标，模型为了过检查牺牲真实质量。
- 工具过多导致 workflow 变慢、变贵、难 debug。
- 没有记录 feedback 和 revision 的 trace，后续无法分析改进是否有效。

## Related Notes

- [Using External Feedback](../notes/module-02-reflection-design-pattern/06-using-external-feedback.md)
- [Code Execution](../notes/module-03-tool-use/06-code-execution.md)
- [What are tools?](../notes/module-03-tool-use/01-what-are-tools.md)
- [Tool use](tool-use.md)
- [Code execution tool](code-execution-tool.md)
- [Reflection pattern](reflection-pattern.md)
- [Agentic evals](agentic-evals.md)
