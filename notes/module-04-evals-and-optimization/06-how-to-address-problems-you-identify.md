---
course: Agentic AI
module: 4
lesson: 6
title: "How to address problems you identify"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/ldf4ci/how-to-address-problems-you-identify
watched_at: 2026-07-14
reviewed_at:
tags: [optimization, component-improvement, prompt-engineering, model-selection, fine-tuning, hyperparameters]
---

# How to Address Problems You Identify

## TL;DR

Error analysis 和 component-level eval 只是告诉你“哪里坏了”；这节回答“发现问题后怎么修”。修法取决于问题 component 的类型：非 LLM component 通常先调参数或替换组件；LLM component 通常先改 prompt、换模型、拆步骤，必要时再 fine-tune。每次改动都要回到 eval，而不是凭感觉判断。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理，只保留结构化总结、设计判断和个人问题，不保存完整逐字稿。

## Core Ideas

- 发现问题后，先判断它是 non-LLM component 还是 LLM component。
- Non-LLM component 的常见修法：tune hyperparameters 或 replace component。
- LLM component 的常见修法：improve prompt、try a new model、split up the step、fine-tune a model。
- Prompt 可以通过更明确 instruction 和 few-shot examples 提升稳定性。
- Model intelligence / instruction following 能力会显著影响结果，不同模型适合不同步骤。
- 拆步骤可以降低单个 LLM call 的难度，让每一步更容易评估和修正。
- Fine-tuning 更适合有内部数据、稳定任务分布、且 prompt/model swap 不够用的场景。
- 选择修法时要用 eval 比较效果，而不是只看单个样例是否变好。

## Lecture Map

1. 接上前几节：error analysis 找到问题 component 后，需要决定怎么修。
2. 对 non-LLM components，优先考虑调参数或替换组件。
3. 对 LLM components，可以从 prompt、model、task decomposition、fine-tuning 四个方向改。
4. 用 PII redaction 例子说明 instruction-following 能力差异：模型可能漏掉 PII 或不按格式输出。
5. 强调要培养对不同模型能力的直觉：多试模型、维护个人 eval set、读别人 prompts。
6. 过渡到后续 latency/cost：不同修法也会影响速度和成本。

## Non-LLM Component Fixes

Non-LLM component 指 workflow 中不是 LLM 本身的步骤，例如：

- web search
- RAG retrieval
- code execution
- OCR / PDF-to-text
- trained ML model, such as speech recognition or object detection

常见修法：

| Component | Tunable Things | Replace With |
| --- | --- | --- |
| Web search | number of results, date range, domain constraints | different search engine/provider |
| RAG retrieval | similarity threshold, chunk size, top-k | different vector DB / retrieval provider |
| ML detector | detection threshold, confidence cutoff | different model |
| Code execution | timeout, package set, resource limit | different sandbox/runtime |

这类问题通常不要先改 prompt，因为瓶颈可能不在 LLM，而在工具、检索、参数或外部系统本身。

## LLM Component Fixes

### 1. Improve Prompt

先尝试让 instruction 更清楚，特别是：

- 明确 output format。
- 明确 success criteria。
- 明确禁止或必须做的事。
- 加入一个或多个 concrete examples，也就是 few-shot prompting。

适合：模型大体能做任务，但格式、边界、细节不稳定。

### 2. Try A New Model

不同模型的能力差异很明显，尤其是：

- instruction following
- reasoning
- extraction accuracy
- tool use reliability
- long-context handling

课程用 PII redaction 例子说明：较弱模型可能漏掉姓名、地址细节，或者没有严格按要求分隔输出；更强模型可能能更完整地识别 PII 并按格式输出。

### 3. Split Up The Step

如果一个 LLM call 同时承担太多任务，可以拆成小步骤：

```text
one hard step
  -> identify entities
  -> classify entities
  -> redact text
  -> validate format
```

拆步骤的好处：

- 每一步更简单。
- 每一步更容易做 component-level eval。
- 错误更容易定位。
- 可以给不同步骤使用不同模型或工具。

代价是 latency、cost 和 orchestration complexity 会增加。

### 4. Fine-Tune A Model

Fine-tuning 适合任务稳定、内部数据足够、且 prompt/model swap 已经不够用的场景。它不是第一反应，因为成本和维护负担更高。

对数据分析 agent 来说，fine-tuning 可能适合：

- 固定业务口径解释。
- 公司内部 SQL 风格。
- 常见图表/报告模板。
- 高频客服/业务问答模式。

但在做 fine-tuning 前，最好先有 eval set，否则很难判断是否真的变好。

## Practical Decision Tree

```text
error analysis 找到问题 component
  -> non-LLM component?
       -> tune hyperparameters
       -> replace provider/tool/model
       -> run component-level eval
  -> LLM component?
       -> improve prompt
       -> try stronger or more suitable model
       -> split step
       -> fine-tune if needed
       -> run component-level eval
  -> then run end-to-end eval
```

## Model Intuition

课程建议持续培养对模型能力的直觉：

- 经常亲手试不同模型。
- 用自己的 eval set 比较模型。
- 阅读别人写得好的 prompts。
- 在同一个 agentic workflow 中，不同步骤可以使用不同模型。
- 用工具如 `aisuite` 快速替换 provider/model 做对照。

这里的重点不是追热门模型，而是弄清楚：哪个模型适合哪个 component。

## Design Takeaways

- “怎么修”要跟 component 类型匹配：工具问题修工具，LLM 问题修 prompt/model/step design。
- 不要默认所有问题都靠 prompt engineering 解决。
- Model swap 是一种合理工程手段，尤其在 instruction following 或 reasoning 明显不足时。
- Split step 是 agentic workflow 的核心能力之一：把一个难任务拆成可评估、可替换的小组件。
- Fine-tuning 应该放在有明确 eval 和足够数据之后，而不是作为第一步。
- 对数据分析 agent，SQL generation、schema retrieval、final explanation 可能需要不同模型和不同 eval。

## Failure Modes

- 发现问题后直接改 prompt，但真正瓶颈在 web search、RAG retrieval 或 database quality。
- 只换更强模型，不分析哪个 component 需要更强能力。
- 拆步骤后没有 eval，导致系统更复杂但不一定更好。
- Fine-tune 前没有 baseline 和 eval，无法证明 fine-tuning 有收益。
- 只看质量，不看 latency/cost，使 workflow 难以上线。
- 为每个问题都换模型，导致系统维护复杂、行为不稳定。

## Implementation Hooks

- Prompting: 建立 prompt versioning，记录每次 instruction / example 的变化。
- Tools: 对 non-LLM tools 暴露可调参数，例如 `top_k`、date range、domain filters、threshold。
- Workflow: 支持把一个 LLM step 拆成多个 component，并记录每个 component trace。
- Memory/state: 保存 model name、prompt version、tool config、eval score，方便回溯。
- Evals: 每种修法都要配 component-level eval；上线前再跑 end-to-end eval。
- Human-in-the-loop: 对 fine-tuning 数据和模型输出做人工抽查，避免把错误模式学进去。

## My Notes

- 这节对我做数据分析 agent 很实用：SQL 错不一定要直接换大模型，可能是 schema retrieval 给错了表；图表差也不一定是 final explanation 问题，可能是 chart spec step 太复杂。
- 我应该把“修法菜单”记住：调参数、换组件、改 prompt、换模型、拆步骤、fine-tune。每次先问问题在哪个 component，再选菜单。
- PII redaction 例子提醒我：instruction following 本身就是一种能力，不同模型差别很大，不能只看 benchmark 名次。

## Open Questions

- 对数据分析 agent，哪些 component 值得用强模型，哪些可以用更便宜/更快的小模型？
- SQL generation 失败时，应该先加 few-shot examples，还是先拆成 schema selection -> SQL draft -> SQL validation？
- Fine-tuning 内部数据分析 agent 前，最小 eval set 应该长什么样？
- 如何把 prompt/model/tool config 的版本和 eval results 绑定起来，方便回滚？

## Related

- [Component-level evaluations](04-component-level-evaluations.md)
- [Component-level eval concept card](../../concepts/component-level-eval.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Error analysis](../../concepts/error-analysis.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 4 README / summary。
- [x] 增加 glossary / open questions。
- [x] 更新 component-level eval / agentic evals 相关概念卡。
- [ ] 后续补整理 ungraded lab: adding a component-level eval to the research workflow。
- [x] 下一节继续 latency, cost optimization。
