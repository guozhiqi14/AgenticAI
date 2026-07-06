---
course: Agentic AI
module: 2
lesson: 6
title: Using External Feedback
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/txqmf0/using-external-feedback
watched_at: 2026-07-06
reviewed_at:
tags: [reflection, external-feedback, tools, regex, fact-checking, word-count]
---

# Using External Feedback

## TL;DR

Reflection 最强的版本不是让 LLM 只靠自己“想一想哪里错了”，而是把外部工具产生的新信息喂回去。代码执行结果、错误日志、pattern matching、web search、trusted sources、word count 都可以成为 external feedback，让 reflection 从“自我感觉”变成“基于新证据修改”。

## Core Ideas

- 纯 prompt engineering 经常会遇到 diminishing returns：prompt 调一阵后，效果提升变慢。
- 加入 reflection 可能带来一段 performance bump，但如果只是让 LLM 看同一份信息自我反思，提升也会有限。
- External feedback 能提供 LLM 原本没有的新信息，因此通常比纯自我 reflection 更有力。
- 软件工具不一定复杂；简单的正则、计数器、代码执行、搜索工具，都可以作为 feedback source。
- 外部反馈的作用不是替代 LLM，而是告诉 LLM “哪里具体不对、证据是什么、应该朝哪个方向改”。

## Performance Curve Intuition

这节课用了一条很实用的直觉：

```text
direct generation + prompt tuning
  -> early improvement
  -> plateau

add reflection
  -> possible bump
  -> tune generation prompt + reflection prompt

add external feedback
  -> stronger signal
  -> higher improvement trajectory
```

简单说：如果你一直调 prompt 但效果不再明显提升，不一定是你不会写 prompt；可能是系统缺少新的 feedback signal。

## External Feedback Examples

### 1. Code Execution Feedback

如果 LLM 写代码，可以运行代码，把 output、error message 或 test result 喂回模型。模型拿到错误日志后，通常比只靠自己 review 更容易修正。

```text
generate code -> run code -> capture output/error -> revise code
```

这和你写 Python 时很像：不是盯着代码想半天，而是先跑一下，看 traceback 或输出结果。

### 2. Competitor Name Checker

如果 LLM 写营销邮件或客服文案时不应该提到竞品名，可以写一个简单工具做 pattern matching。比如用 regular expression 或字符串匹配扫描输出，发现竞品名后，把具体命中的词反馈给 LLM，让它重写。

```text
draft email -> scan competitor names -> feedback: "contains X" -> rewrite
```

这个工具不需要很智能，但 feedback 很明确。

### 3. Fact-checking With Trusted Sources

如果 LLM 写 research essay 或事实性内容，可以用 web search 或 trusted sources 查证。比如模型对某个历史事实表达不准确时，把检索到的更精确信息作为 external feedback，再让模型修订文本。

核心不是“模型自己再想一次”，而是给它新的 evidence。

### 4. Word Count Tool

LLM 对精确字数限制并不稳定。如果你要求摘要必须少于 150 words，可以写一个 word count tool。超过限制时，把实际字数反馈给模型，让它重写。

```text
draft abstract -> count words -> feedback: "182 words, limit is 150" -> revise
```

这类问题很适合用代码解决，因为计数是确定性的。

## Design Takeaways

- 当 prompt tuning plateau 时，优先想：能不能加一个外部 feedback source？
- External feedback 不一定要是复杂工具；能稳定指出具体问题的小工具就很有价值。
- 对确定性规则，优先用代码检查，比如字数、禁用词、JSON schema、SQL 执行错误。
- 对事实性问题，优先用检索或可信来源作为 feedback，而不是要求 LLM 自己判断自己是否正确。
- Feedback 越具体，revision 越容易有效。`make it better` 不如 `output contains competitor name X`。
- External feedback 是 reflection 和 tool use 之间的桥：下一模块的 tool use 会把这种能力系统化。

## Failure Modes

- Feedback tool 设计太粗，只返回“bad”，没有告诉模型哪里错。
- 外部来源不可信，把错误证据喂给模型，反而让输出变差。
- Regex 或 pattern matching 过于简单，漏掉变体或误伤正常词。
- 只检查容易自动化的指标，比如字数，却忽略真实质量。
- 工具调用太多，latency 和 cost 上升，但没有 eval 证明质量提升。

## Implementation Hooks

- Code execution: 捕获 stdout、stderr、traceback、unit test result。
- Pattern matching: 检查禁用词、竞品名、敏感词、格式违规。
- Retrieval/search: 为事实性文本提供 evidence snippet 或 trusted source。
- Deterministic checks: word count、JSON validity、schema validation、SQL execution。
- Feedback format: 尽量结构化，比如 `{issue, evidence, severity, suggested_fix}`。
- Evals: 比较 no feedback、self-reflection、external-feedback reflection 三种 workflow。

## My Questions

- 我的数据分析 agent 里，最容易做 external feedback 的工具是什么：SQL 执行、schema check、指标口径检查、还是结果 sanity check？
- 对事实性分析报告，trusted sources 应该来自哪里：数据表元信息、业务 wiki、历史 SQL、还是人工标注规则？
- 哪些 feedback 应该自动进入 revision，哪些 feedback 应该触发 human review？
- 如何避免系统为了满足简单工具检查，反而牺牲真实表达质量？

## Related

- [Reflection to Improve Outputs of a Task](01-reflection-to-improve-outputs-of-a-task.md)
- [Evaluating the Impact of Reflection](05-evaluating-the-impact-of-reflection.md)
- [Reflection pattern](../../concepts/reflection-pattern.md)
- [External feedback](../../concepts/external-feedback.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 新增 external feedback concept card。
- [x] 更新 reflection pattern。
- [x] 更新 Module 2 summary / README。
- [ ] 后续 SQL reflection lab 时，记录 SQL execution error 如何作为 external feedback。
- [ ] 为未来 labs 设计 `no feedback -> self-reflection -> external feedback` 三组对照。
