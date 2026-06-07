---
course: Agentic AI
module: 1
lesson: 7
title: "Evaluating Agentic AI (Evals)"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/46yh5j/evaluating-agentic-ai-(evals)
watched_at: 2026-05-29
reviewed_at:
tags: [evals, error-analysis, llm-as-judge, traces, workflow-iteration]
---

# Evaluating Agentic AI (Evals)

## TL;DR

能不能做好 agentic workflow，很大程度取决于能不能建立 disciplined evaluation process。实践上通常不是一开始就预测所有错误，而是先构建 workflow，检查输出和中间 traces，找出不满意的模式，再把这些问题转成 eval 来持续改进。

## Core Ideas

- Evals 是改进 agentic workflow 的核心工具，不只是最后打分。
- 很多错误很难提前预判，所以要先看真实 outputs，再做 error analysis。
- 对 objective criteria，可以写代码检查，比如是否提到竞争对手。
- 对 subjective criteria，可以用 LLM-as-judge，但简单的 1-5 打分并不总可靠。
- Agentic AI evals 主要分两类：end-to-end evals 和 component-level evals。
- 观察 intermediate outputs / traces 是定位问题的关键。

## Evaluation Workflow

一个实用流程：

1. Build an initial workflow.
2. Run it on realistic examples.
3. Manually inspect outputs and traces.
4. Identify recurring failure patterns.
5. Turn failure patterns into evals.
6. Improve prompts, tools, decomposition, or models.
7. Re-run evals and compare progress.

这和上一节 task decomposition 连得很紧：先把 workflow 拆成步骤，才有可能分别观察和评估每一步。

## Example: Competitor Mentions

课程用 customer order inquiry agent 举例。系统可能意外地在回复里提到竞争对手，例如把自家服务和 competitor 做比较。

这类问题很难在构建前完全预判，但一旦你读到输出，就可以把它变成 objective eval：

- 准备竞争对手名单。
- 扫描 agent 输出是否包含这些名字。
- 统计错误发生频率。
- 修改系统后持续观察这个频率是否下降。

这个例子说明：好的 eval 往往来自实际失败，而不是凭空设计。

## Objective vs Subjective Evals

| Type | What It Means | Example | How To Evaluate |
| --- | --- | --- | --- |
| Objective eval | 判断标准相对明确，能写代码检查 | 是否提到 competitor、是否包含必填字段、是否调用了错误工具 | Code-based checks |
| Subjective eval | 判断标准更像质量判断 | 研究报告是否深入、客服回复是否礼貌、答案是否清晰 | LLM-as-judge 或人工评审 |

LLM-as-judge 可以用于主观质量评估，但要谨慎。简单让模型给 1-5 分可以作为初步方法，但课程提醒这种打分并不总是很可靠，后续模块会讲更好的评分技巧。

## End-to-End vs Component-Level Evals

| Eval Type | Focus | Useful For |
| --- | --- | --- |
| End-to-end eval | 整个 agent 的最终输出质量 | 判断用户看到的结果是否变好 |
| Component-level eval | 单个 workflow step 的输出质量 | 定位问题、改进某个步骤、避免只看最终结果 |

Component-level eval 对 agentic workflow 特别重要，因为一个复杂系统可能是某个中间步骤出了问题：提取错、query 生成差、工具选错、检索结果差、draft 不合格等。

## Traces And Error Analysis

Trace 指的是 agentic workflow 每一步的中间输出和动作记录，例如：

- LLM 生成了什么 search query。
- 调用了哪个 tool。
- tool 返回了什么。
- 中间判断是什么。
- 最终答案是如何生成的。

Error analysis 就是读这些 traces，找出系统没有达到预期的地方。没有 traces，就很容易只看到 final answer 坏了，却不知道坏在哪里。

## Design Takeaways

- 先让系统跑起来，再从真实失败中设计 eval。
- 不要只评估 final answer；agentic workflow 需要评估中间步骤。
- Objective eval 优先用代码，因为更稳定、便宜、可重复。
- Subjective eval 可以用 LLM-as-judge，但要注意 judge 本身也可能不稳定。
- 每次改 workflow、prompt、tool 或 model，都应该能通过 eval 判断是否真的变好。
- Logging/tracing 不是可选项，而是 eval 和 error analysis 的基础设施。

## Failure Modes

- 只凭主观感觉改 prompt，没有可重复 eval。
- 只看最终输出，不看 intermediate traces，导致无法定位问题。
- 过早设计复杂 eval，但没有基于真实失败。
- 对主观任务只用简单 1-5 分，误以为分数很精确。
- Component eval 缺失，导致每次系统变差都只能大范围猜原因。
- Eval 指标变好，但用户真实体验没有变好，说明 eval 没抓住核心质量。

## Implementation Hooks

- Dataset: 收集一组真实或接近真实的输入样例。
- Trace logging: 保存每一步 input、output、tool call、latency、cost、error。
- Objective checks: 用代码检查格式、禁用词、必填字段、工具调用、schema validity。
- LLM-as-judge: 用于主观质量判断，但尽量给明确 rubric，而不是只要 1-5 分。
- Component evals: 给 extraction、query generation、retrieval、tool selection、drafting 等关键步骤分别建 eval。
- Regression tracking: 每次修改 workflow 后，比较 eval 是否改善或回退。

## My Questions

- 对我自己的 agent 项目，哪些 failure pattern 应该先转成 eval？
- Component-level eval 应该覆盖多少步骤，才不会过度工程化？
- LLM-as-judge 的 rubric 应该怎么写，才能比 1-5 分更稳定？
- 如何避免 eval 指标被优化了，但真实用户体验没有变好？

## Related

- [Task Decomposition](06-task-decomposition-identifying-the-steps-in-a-workflow.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 agentic evals concept card。
- [ ] 后续 Module 4 学 evals 时，回头扩展这篇。
- [ ] 给未来 labs 加一个 `evals.md` 模板或 checklist。

