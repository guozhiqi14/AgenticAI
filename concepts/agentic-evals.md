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
6. Improve workflow.
7. Re-run evals and compare.

## Eval Types

| Type | Use When | Example |
| --- | --- | --- |
| Objective code eval | 标准清晰、能自动检查 | 禁止提到 competitor、JSON schema 是否 valid、必填字段是否存在 |
| LLM-as-judge | 标准主观、需要质量判断 | 文章是否深入、回复是否礼貌、答案是否完整 |
| End-to-end eval | 关心最终用户体验 | 最终客服回复是否可发送 |
| Component-level eval | 需要定位某一步问题 | query generation、retrieval、field extraction、tool selection |

## Traces

Trace 是 agentic workflow 的中间执行记录，包括每一步 input/output、tool call、retrieved context、model decision、latency、cost 和 error。

没有 traces，eval 很容易停留在“结果好/不好”；有 traces，才能知道应该改哪一步。

## Design Questions

- 我现在最常见的失败模式是什么？
- 这个失败能不能用代码客观检查？
- 如果需要 LLM-as-judge，rubric 是否足够明确？
- 应该评估 end-to-end，还是某个 component？
- 修改 workflow 后，如何判断没有引入 regression？
- Eval 指标是否真的代表用户关心的质量？

## Practical Rule

先从真实失败中建立小而明确的 eval。优先用 objective checks；主观质量再用 LLM-as-judge，并配合清晰 rubric 和人工抽查。

## Related Notes

- [Evaluating Agentic AI](../notes/module-01-introduction-to-agentic-workflows/07-evaluating-agentic-ai-evals.md)
- [Task decomposition](task-decomposition.md)

