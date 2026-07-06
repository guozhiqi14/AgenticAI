---
course: Agentic AI
module: 2
lesson: 5
title: Evaluating the Impact of Reflection
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/vtzr25/evaluating-the-impact-of-reflection
watched_at: 2026-07-06
reviewed_at:
tags: [reflection, evals, sql, llm-as-judge, rubric, prompt-iteration]
---

# Evaluating the Impact of Reflection

## TL;DR

Reflection 会增加一次或多次额外步骤，所以不能只因为“听起来更高级”就默认保留。应该用 eval 比较 `without reflection` 和 `with reflection`：如果任务有明确正确答案，用 objective eval；如果质量更主观，比如图表好不好，用 rubric-based LLM-as-judge，而不是简单让 LLM 在两个结果里二选一。

## Core Ideas

- Reflection 通常会提升质量，但也会增加 latency 和 cost。
- 判断 reflection 是否值得，需要先建立 direct generation / no-reflection baseline。
- 对 SQL/database query 这类有正确答案的任务，可以准备 prompts + ground truth answers，然后用代码统计正确率。
- 对图表这类主观任务，可以用 multimodal LLM-as-judge，但最好让它按明确 rubric 对单个结果逐项打分。
- 让 LLM 直接比较 A/B 两个结果容易出现 position bias：模型可能偏向第一个或第二个选项。
- Rubric 最好拆成多个 binary criteria，比如标题是否清楚、坐标轴标签是否存在、图表类型是否合适。
- 有了 eval set 之后，就可以系统比较不同 generation prompt 和 reflection prompt，而不是凭感觉调 prompt。

## Example 1: SQL Query Reflection

课程用一个零售店数据库问题做例子：用户问“哪种颜色产品总销售额最高？”或“2025 年 5 月卖出了多少件商品？”系统可以让 LLM 先生成 database query，再让另一个 LLM reflect / revise query，最后执行 query 得到答案。

如果你不熟 SQL，可以先把它理解成 pandas 查询：

```python
# pandas 里你可能会这样筛选、聚合、排序
df.query("month == '2025-05'").groupby("color")["sales"].sum()
```

SQL 做的是类似事情，只是它面向数据库表。这里的关键不是 SQL 语法本身，而是：query 的结果可以和 ground truth answer 对比，所以它适合 objective eval。

评估流程：

1. 准备一批真实或代表性问题。
2. 给每个问题写 ground truth answer。
3. 跑 no-reflection workflow：第一次生成的 query 直接执行。
4. 跑 reflection workflow：先生成 query，再反思修正，再执行。
5. 比较两组最终答案的正确率。

课程示例里，no reflection 正确率约为 87%，with reflection 约为 95%。这说明在这个例子中，reflection 对 query quality 有实际帮助。

## Example 2: Chart Evaluation

图表质量更主观。比如上一节 chart generation workflow 里，reflection 之后的图表看起来更好，但“更好”不一定能用一个简单正确答案判断。

一个直接做法是把两张图交给 multimodal LLM，让它判断哪张更好。但课程提醒：pairwise comparison 不太稳定，容易受 prompt wording 和 presentation order 影响。

更稳的做法是 rubric-based scoring：

```text
Given one chart image, evaluate it against these binary criteria:
- Does it have a clear title?
- Are axis labels present?
- Is the chart type appropriate?
- Is the plot easy to read?
- Does it answer the user's question?
```

每条标准给 0/1 分，再加总。相比让模型直接给 1-5 分，多个 binary criteria 通常更可控，也更容易定位到底哪里差。

## Objective vs Subjective Evals

| Eval Type | Best For | How To Use |
| --- | --- | --- |
| Objective code eval | 有明确正确答案或格式规则 | 用代码比对答案、schema、字段、数值 |
| Rubric-based LLM-as-judge | 图表、文案、解释质量等主观任务 | 把质量拆成具体 criteria，逐项打分 |

优先用 objective eval，因为它更便宜、更稳定、更容易复现。只有当任务质量确实主观时，再用 LLM-as-judge，并且要给清晰 rubric。

## Design Takeaways

- Reflection 是否值得，应该通过质量提升 vs latency/cost 来判断。
- Eval set 不需要一开始很大；10 到 15 个代表性 examples 就能帮助比较 prompt 方向。
- 对每次 prompt 改动，都应该能重新跑 eval，看是否真的提升。
- 不要让 LLM judge 只做“哪个更好”的 pairwise choice；改成按 rubric 评价单个 output。
- Binary rubric 比模糊的 1-5 分更容易校准，也更容易解释。
- Reflection eval 不只看 final output，也可以单独看 intermediate artifact，比如 SQL query、chart spec、critique text。

## Failure Modes

- 没有 no-reflection baseline，无法证明 reflection 是否有价值。
- Eval examples 太少或太偏，导致 prompt 只对小样本过拟合。
- Objective eval 写错 ground truth，误导后续 prompt tuning。
- LLM-as-judge 直接比较 A/B，受到 position bias 影响。
- Rubric 太抽象，比如只写“quality”或“looks good”，导致 judge 输出不稳定。
- Eval 分数提升，但实际用户体验没有提升。

## Implementation Hooks

- Baseline: 保存 no-reflection 和 with-reflection 的结果。
- Dataset: 建一个小的 prompts + ground truth / expected criteria 表。
- Metrics: objective 任务看 accuracy；主观任务看 rubric score 和人工抽查一致性。
- Prompt iteration: 每次修改 generation prompt 或 reflection prompt 后 rerun eval。
- Logging: 保存 v1 output、critique、revised output、final answer、eval result。
- Judge prompt: 用 binary criteria，避免只让 judge 直接偏好排序。

## My Questions

- 对我自己的数据分析 agent，哪些问题可以先做 objective eval？
- 如果要评估图表或分析报告，rubric 应该由谁定义：我、用户、业务方，还是先从失败案例里归纳？
- 小 eval set 多大才够？什么时候需要从 10-15 个 examples 扩展到更多？
- 如何发现 eval 分数提升但真实体验没提升的情况？

## Related

- [Reflection to Improve Outputs of a Task](01-reflection-to-improve-outputs-of-a-task.md)
- [Why Not Just Direct Generation?](02-why-not-just-direct-generation.md)
- [Reflection pattern](../../concepts/reflection-pattern.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 2 README。
- [x] 补充 reflection pattern 的 eval 判断。
- [x] 补充 agentic evals 对 rubric-based LLM-as-judge 的说明。
- [x] 后续 `Using external feedback` 之后，补充 external feedback 如何增强 reflection eval。
- [ ] 未来 lab 建立 direct vs reflection 的最小 SQL 或 chart eval。
