# Module 2: Reflection Design Pattern

## Lesson Checklist

- [x] 01 - Reflection to improve outputs of a task
- [x] 02 - Why not just direct generation?
- [ ] 03 - Chart generation workflow
- [ ] 04 - Ungraded Lab: Chart Generation
- [x] 05 - Evaluating the impact of reflection
- [x] 06 - Using external feedback
- [ ] 07 - Ungraded Lab: Improving SQL Generation with Reflection
- [ ] 08 - Module 2 quiz
- [ ] 09 - M2 Graded Lab

## Module-Level Themes

- Reflection 是一种简单但常有收益的 agentic design pattern。
- Reflection 的核心是 draft -> critique/feedback -> revise。
- Reflection 不等于魔法；它通常带来 modest bump，而不是保证正确。
- Reflection 是否值得保留，要用 eval 比较 no-reflection baseline 和 reflection workflow。
- Objective eval 适合 SQL/database query 这类有正确答案的任务；主观任务需要 rubric-based LLM-as-judge。
- 外部反馈会显著增强 reflection，比如代码运行结果、错误日志、测试结果、用户反馈。
- External feedback 可以来自简单工具：regex/pattern matching、word count、代码执行、web search、trusted sources。
- 不同模型可以负责不同角色，比如 general model 写初稿，reasoning model 找 bug。

## Notes

- [01 - Reflection to improve outputs of a task](01-reflection-to-improve-outputs-of-a-task.md)
- [02 - Why not just direct generation?](02-why-not-just-direct-generation.md)
- [05 - Evaluating the impact of reflection](05-evaluating-the-impact-of-reflection.md)
- [06 - Using external feedback](06-using-external-feedback.md)
