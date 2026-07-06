# Concept: Reflection Pattern

## Definition

Reflection pattern 是一种 agentic design pattern：让 LLM 生成初稿，再让同一个或另一个 LLM 检查、批评、修正输出。它的基本结构是 draft -> critique/feedback -> revise。

## Why It Matters

Reflection 是最容易实现的 agentic workflow 之一。它可以在不引入复杂 tool use 或 planning 的情况下，让系统通过自我检查或外部反馈提升输出质量。

## Basic Shape

```text
Input
  -> Generate draft
  -> Critique or reflect
  -> Revise
  -> Final output
```

更强的版本会加入 external feedback：

```text
Input
  -> Generate draft/code/query
  -> Run/check/test
  -> Feed output/errors back to LLM
  -> Revise
```

## External Feedback

External feedback 是 reflection 的重要增强版：它把来自 LLM 之外的新信息加入 critique/revise loop。

常见 feedback source：

- Code execution: output、error message、test result。
- Pattern matching: 竞品名、禁用词、敏感词。
- Retrieval/search: trusted source、事实证据。
- Deterministic checks: word count、schema validation、SQL execution。
- User feedback: 用户明确指出的问题或偏好。

这类 feedback 往往比纯自我反思更强，因为模型拿到了原本没有的新 evidence。

## Direct Generation Baseline

使用 reflection 前，最好先建立 direct generation / zero-shot baseline：

```text
Input -> LLM -> Final output
```

Reflection 的价值不是“看起来更 agentic”，而是相对 baseline 是否真的提升质量。常见比较维度包括 error rate、format validity、completeness、human preference、latency 和 token cost。

## Evaluating Reflection

Reflection 会带来额外 LLM call，所以需要用 eval 判断是否值得：

| Task Type | Eval Approach | Example |
| --- | --- | --- |
| 有明确正确答案 | Objective eval / code-based eval | SQL query 得到的答案是否等于 ground truth |
| 质量主观 | Rubric-based LLM-as-judge | 图表是否有清晰标题、坐标轴标签、合适图表类型 |
| 有外部检查信号 | Tool/checker feedback eval | no feedback vs external-feedback reflection |

不要只让 judge model 直接比较两个 outputs 哪个更好；这种 pairwise comparison 可能受 position bias 影响。更稳的做法是给单个 output 一组 binary criteria，然后把分数加总。

## Best Fit

- 代码生成和修复。
- 文案、email、文章 draft。
- SQL/query generation。
- 图表、报告、结构化输出。
- 有明确 rubric 或外部检查信号的任务。
- 复杂 structured output，例如嵌套 JSON 或格式要求严格的 HTML。
- 需要检查完整性、语气、事实、负面含义或可读性的任务。

## Design Questions

- 初稿和 critique 是否应该由同一个模型完成？
- 有没有外部 feedback 可以加入？
- Critique rubric 是否足够具体？
- 反思后如何判断结果真的变好？
- 应该最多迭代几轮？
- Reflection 的质量收益是否值得额外 latency 和 cost？
- Direct generation baseline 表现如何？
- 哪些 criteria 最值得放进 reflection prompt？
- Reflection 的 eval 应该是 objective code check，还是 rubric-based judge？
- 是否可以加入一个小工具，让 feedback 更具体？

## Failure Modes

- 模型只是重述初稿，没有真正发现问题。
- Critique 太泛，revision 没有方向。
- 没有外部反馈时，模型可能自信地修错。
- 外部反馈本身不可靠时，revision 会被错误证据带偏。
- 循环太多，成本和 latency 失控。
- Reflection 让表面文字更漂亮，但事实或逻辑没有改善。

## Practical Rule

优先在有可检查输出或外部反馈的任务上使用 reflection。没有反馈时，至少给 critique step 一个明确 rubric；有反馈时，把 error logs、test results、user comments 或 retrieved evidence 作为 reflection input。

不要只写 `make it better`。更好的 reflection prompt 应该明确要求 review/reflect，并列出检查标准，例如 format、completeness、tone、facts、negative connotations、pronunciation 或 correctness。

## Related Notes

- [Reflection to Improve Outputs of a Task](../notes/module-02-reflection-design-pattern/01-reflection-to-improve-outputs-of-a-task.md)
- [Why Not Just Direct Generation?](../notes/module-02-reflection-design-pattern/02-why-not-just-direct-generation.md)
- [Evaluating the Impact of Reflection](../notes/module-02-reflection-design-pattern/05-evaluating-the-impact-of-reflection.md)
- [Using External Feedback](../notes/module-02-reflection-design-pattern/06-using-external-feedback.md)
- [External feedback](external-feedback.md)
- [Agentic design patterns](agentic-design-patterns.md)
- [Agentic evals](agentic-evals.md)
