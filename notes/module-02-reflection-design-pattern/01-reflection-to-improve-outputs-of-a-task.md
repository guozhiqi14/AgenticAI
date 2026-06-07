---
course: Agentic AI
module: 2
lesson: 1
title: Reflection to Improve Outputs of a Task
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/shknq1/reflection-to-improve-outputs-of-a-task
watched_at: 2026-06-07
reviewed_at:
tags: [reflection, external-feedback, critique, revision, design-patterns]
---

# Reflection to Improve Outputs of a Task

## TL;DR

Reflection 是让 LLM 先生成一个 draft，再让同一个或另一个 LLM 检查、批评、修正这个 draft。它不保证 100% 正确，但经常能带来性能提升；如果能把外部反馈加入 reflection，比如代码执行结果或错误日志，效果通常更强。

## Core Ideas

- 人类常常通过“写初稿 -> 回看问题 -> 修改”来提升输出，LLM 也可以用类似流程。
- 最简单的 reflection workflow 是 hard-coded 两步：先生成 v1，再把 v1 交给 LLM 反思并生成 v2。
- Reflection 可以用于 email、code、文章、图表、SQL 等很多输出类型。
- 同一个模型可以负责生成和反思；也可以用不同模型分工。
- Reasoning/thinking models 往往更适合找 bug、检查逻辑、发现缺陷。
- Reflection 最有价值的时候，是它能获得额外外部信息，比如运行代码后的 error message。

## Basic Workflow

```text
task -> LLM generates draft v1
draft v1 -> LLM critiques / reflects
critique -> LLM revises into v2
```

如果有外部反馈，可以变成：

```text
task -> LLM generates code v1
code v1 -> execute / test
execution result or error log -> LLM reflects
reflection -> LLM writes code v2
```

第二种通常更强，因为模型不是只靠自己“想哪里可能错”，而是拿到了新信息。

## Examples From The Lesson

### Email Revision

一个快速写出的 email draft 可能有这些问题：

- 时间不够具体。
- 有 typo。
- 忘记署名。

Reflection step 可以让 LLM 读 v1，指出问题，并生成更清晰、更完整的 v2。

### Code Improvement

LLM 先生成一版代码，然后另一次 LLM call 检查 bug、style、efficiency，并写出改进版。

如果能运行代码，错误日志会变成非常强的 feedback signal。比如 v1 运行后出现 syntax error，把错误日志交给 LLM，它就更容易定位问题并生成 v2。

## Design Takeaways

- Reflection 是最容易实现的 agentic pattern 之一，因为它可以只是多一次 LLM call。
- Reflection 最适合有“可检查 output”的任务，比如代码、文案、答案草稿、SQL、图表说明。
- 仅让模型自我反思有帮助，但外部反馈更重要。
- 对代码类任务，运行结果、unit tests、error logs 是很好的 reflection input。
- 不同模型可以扮演不同角色：便宜模型生成初稿，reasoning model 做 critique。
- Reflection 的收益要用 eval 验证，不要默认每次反思都会更好。

## Failure Modes

- LLM 没有外部反馈，只是重复自己原来的假设。
- Critique prompt 太泛，导致反馈空泛，无法指导 revision。
- Reflection loop 没有 stopping condition，成本和 latency 增加。
- 反思后输出更自信，但其实没有变正确。
- 对不容易验证的任务，模型可能无法判断自己哪里错。

## Implementation Hooks

- Prompting: 给 critique step 明确检查维度，例如 correctness、style、efficiency、missing details。
- External feedback: 尽量把 test result、error log、lint output、user feedback、retrieved evidence 加入 reflection。
- Model routing: 初稿和 critique 可以用不同模型。
- Evals: 比较 direct generation vs reflection workflow 的质量、成本和 latency。
- Logging: 保存 v1、critique、external feedback、v2，方便 error analysis。
- Stopping: 限制 reflection iterations，或者用 eval/threshold 判断是否继续。

## My Questions

- 哪些任务最适合用 reflection，哪些任务反思收益很小？
- Critique prompt 应该如何写，才能避免空泛建议？
- Reflection 的迭代次数如何控制？
- 对没有明确外部反馈的任务，如何判断 reflection 是否真的提升？

## Related

- [Agentic Design Patterns](../module-01-introduction-to-agentic-workflows/08-agentic-design-patterns.md)
- [Evaluating Agentic AI](../module-01-introduction-to-agentic-workflows/07-evaluating-agentic-ai-evals.md)
- [Reflection pattern](../../concepts/reflection-pattern.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 reflection pattern concept card。
- [x] 下一节比较 reflection vs direct generation 后，补充适用条件。
- [ ] 后续 chart/SQL lab 后，补充真实 workflow 和 eval 观察。
