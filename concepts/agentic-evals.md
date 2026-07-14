# Concept: Agentic Evals

## Definition

Agentic evals 是用来评估 agentic workflow 是否有效的测试和分析方法。它既包括整个系统最终输出的 end-to-end eval，也包括每个 workflow step 的 component-level eval。

## Why It Matters

Agentic workflow 往往包含多个 LLM calls、tools、retrieval、API 和决策点。只看最终答案，很难知道问题出在哪里。Evals 和 traces 能把系统改进从“凭感觉调 prompt”变成更可控的工程过程。

## Core Loop

1. Build initial workflow.
2. Run realistic examples.
3. Inspect outputs and traces.
4. Identify recurring failures.
5. Create evals for those failures.
6. Improve the right component with a matching repair method.
7. Re-run component-level evals and compare.
8. Re-run end-to-end evals before trusting the change.

这个 loop 不是线性瀑布。实际开发中会在 build 和 analyze 之间来回切换：先快速搭出 end-to-end workflow，再读 outputs/traces，然后逐步加入 eval set、error analysis、component-level evals 和 latency/cost measurement。

## Start From Observed Failures

课程反复强调：agentic workflow 很难在构建前准确预测哪里会失败。更实用的方式是先做一个安全、可观察的 quick prototype，然后人工阅读一批 outputs/traces，找出 recurring failure modes。

例如：

- invoice processor 经常混淆 invoice date 和 due date。
- marketing caption assistant 经常超过 10 words。
- research agent 有时漏掉专家会提到的关键观点。

这些真实失败会告诉你最该优先评估什么。Eval 不是为了“看起来工程化”而存在，而是为了把最重要的 failure mode 变成可跟踪的改进信号。

## Eval Types

| Type | Use When | Example |
| --- | --- | --- |
| Objective code eval | 标准清晰、能自动检查 | 禁止提到 competitor、JSON schema 是否 valid、必填字段是否存在 |
| LLM-as-judge | 标准主观、需要质量判断 | 文章是否深入、回复是否礼貌、答案是否完整 |
| End-to-end eval | 关心最终用户体验 | 最终客服回复是否可发送 |
| Component-level eval | 需要定位某一步问题 | query generation、retrieval、field extraction、tool selection |

## End-To-End vs Component-Level

End-to-end eval 评估完整 workflow 的最终输出，适合确认用户体验是否真的变好。Component-level eval 评估某个单独组件，适合在 error analysis 已经定位瓶颈后，用更快、更清晰的信号优化这个组件。

一个常见流程是：

```text
end-to-end eval -> error analysis -> component-level eval -> component improvement -> end-to-end regression check
```

Component-level eval 不能替代 end-to-end eval。它负责加速局部优化，最终仍要回到整体系统确认局部优化是否真的提升最终质量。

发现问题以后，改法也应该接在 eval 后面：如果是 web search、RAG retrieval、OCR、code execution 或单独训练的 ML model，先考虑调参数或替换组件；如果是 LLM step，先考虑 prompt、model、step split，最后才考虑 fine-tuning。

## End-To-End Eval Design Matrix

Module 4 里给了一个很实用的二维框架：

| | With per-example ground truth | Without per-example ground truth |
| --- | --- | --- |
| Objective code eval | 每张 invoice 的 due date 是否等于人工标注日期 | 每条 marketing caption 是否不超过统一 word limit |
| LLM-as-judge | Research essay 是否覆盖该 topic 的 gold standard talking points | Chart/report 是否满足同一套通用 rubric |

两个判断问题：

- 这个质量标准能不能用代码稳定判断？
- 每个样例是否需要自己的正确答案或 reference label？

### Per-Example Ground Truth

当每个 input 都有不同正确答案时，需要 per-example ground truth。例如每张 invoice 的 due date 不一样，每个 research topic 的 key points 也不一样。

### No Per-Example Ground Truth

有些 eval 不需要为每个样例标答案，因为标准对所有样例都一样。例如 caption 都不能超过 10 words，或者所有 chart 都用同一套 clarity rubric。

## Reflection Evals

评估 reflection 时，先比较 no-reflection baseline 和 reflection workflow：

```text
same prompt set
  -> no reflection result
  -> with reflection result
  -> compare quality, latency, and cost
```

如果任务有明确答案，比如 SQL/database query，可以准备 prompts + ground truth answers，用代码统计 accuracy。

如果任务更主观，比如图表或报告质量，可以用 LLM-as-judge，但最好把 rubric 拆成多个 binary criteria，而不是只让模型直接在两个 outputs 之间选一个。

Pairwise comparison 容易受到 position bias 影响：有些 judge model 会偏向先出现的选项。因此，rubric-based single-output scoring 往往更稳定，也更容易解释。

当 reflection 引入 external feedback 时，可以多比较一组：

```text
no reflection
self-reflection only
reflection with external feedback
```

这样能判断提升来自“多想一步”，还是来自工具/检索/测试带来的新信息。

## Traces

Trace 是 agentic workflow 的中间执行记录，包括每一步 input/output、tool call、retrieved context、model decision、latency、cost 和 error。

没有 traces，eval 很容易停留在“结果好/不好”；有 traces，才能知道应该改哪一步。

Error analysis 会重点阅读失败样例的 traces，把 final output 的问题归因到具体 component。这个过程最好先人工做，等错误标签稳定后，再转成 component-level eval 或 dashboard。

当 workflow quality 已经达标后，trace 里的 latency/cost 字段也会变成优化依据：按 step 看耗时和成本，决定应该优化慢/贵的 component，而不是平均用力。

## Design Questions

- 我现在最常见的失败模式是什么？
- 这个 eval 是从真实 output/trace 的问题中长出来的吗？
- 这个失败能不能用代码客观检查？
- 如果需要 LLM-as-judge，rubric 是否足够明确？
- 每个样例是否需要 per-example ground truth？
- 当前 eval set 是否覆盖了最重要的 10 到 20 个现实样例？
- Judge 是否存在 position bias？是否应该改成 single-output rubric scoring？
- 如果加入 external feedback，是否应该单独评估 feedback source 的质量？
- 应该评估 end-to-end，还是某个 component？
- 如果评估 component，这个局部 metric 是否和 end-to-end quality 相关？
- 找到失败 component 后，当前修法是否匹配 component 类型？
- 这次改动是在提升真实能力，还是只是在迎合当前 eval set？
- 修改 workflow 后，如何判断没有引入 regression？
- 当前 workflow 是否已经到了值得优化 latency/cost 的阶段？
- 哪个 component 是 total latency 或 total cost 的最大 contributor？
- Eval 指标是否真的代表用户关心的质量？
- 如果 metric 和 human/expert judgment 冲突，是 eval 设计错了，还是 workflow 真的没变好？

## Practical Rule

先从真实失败中建立小而明确的 eval。10 到 20 个样例足够启动第一轮迭代；之后随着系统暴露新错误，再扩充 eval set。优先用 objective checks；主观质量再用 LLM-as-judge，并配合清晰 rubric 和人工抽查。

不要把现成 observability 工具当作 eval 本身。Trace/logging/cost tools 能帮你看见系统运行，但 custom evals 才定义“这个具体应用到底哪里做错了、什么算好”。

## Related Notes

- [Evaluating Agentic AI](../notes/module-01-introduction-to-agentic-workflows/07-evaluating-agentic-ai-evals.md)
- [Evaluations (evals)](../notes/module-04-evals-and-optimization/01-evaluations-evals.md)
- [Component-level eval](component-level-eval.md)
- [Error analysis](error-analysis.md)
- [How to Address Problems You Identify](../notes/module-04-evals-and-optimization/06-how-to-address-problems-you-identify.md)
- [Latency/cost optimization](latency-cost-optimization.md)
- [Agentic development loop](agentic-development-loop.md)
- [Evaluating the Impact of Reflection](../notes/module-02-reflection-design-pattern/05-evaluating-the-impact-of-reflection.md)
- [Using External Feedback](../notes/module-02-reflection-design-pattern/06-using-external-feedback.md)
- [Task decomposition](task-decomposition.md)
