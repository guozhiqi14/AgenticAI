---
course: Agentic AI
module: 2
lesson: 2
title: Why Not Just Direct Generation?
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/uz37v2/why-not-just-direct-generation%3F
watched_at: 2026-06-07
reviewed_at:
tags: [reflection, direct-generation, zero-shot, prompting, evals]
---

# Why Not Just Direct Generation?

## TL;DR

Direct generation / zero-shot prompting 是让 LLM 一次性直接生成答案；reflection workflow 则让模型先生成初稿，再按明确 criteria 检查并改进。研究和实践都显示 reflection 经常能提升直接生成的 baseline，但是否值得使用仍要看具体任务、质量收益、成本和 latency。

## Core Ideas

- Direct generation 是最简单的 baseline：给 LLM 一个 instruction，让它一次生成结果。
- Zero-shot prompting 指 prompt 中没有提供 desired input-output examples。
- One-shot / few-shot prompting 是在 prompt 中加入一个或多个示例。
- Reflection 经常能在 direct generation 之上提升表现，尤其是输出有可检查标准时。
- Reflection prompt 的质量很关键：要明确告诉模型“review/reflect”，并给出具体 criteria。
- Reflection 不一定对所有任务都有明显帮助，应该用 eval 比较 direct baseline 和 reflection workflow。

## Direct Generation vs Reflection

| Approach | Workflow | Strength | Risk |
| --- | --- | --- | --- |
| Direct generation / zero-shot | instruction -> final answer | 简单、快、成本低 | 容易漏步骤、格式错、没有自检 |
| Reflection | instruction -> draft -> review/criteria check -> revised answer | 更可能发现错误和补缺 | 多一次或多次 LLM call，增加 latency/cost |

Direct generation 是必须先建立的 baseline。Reflection 的价值要通过和这个 baseline 比较来判断。

## What Zero-Shot Means

Zero-shot prompting 指 prompt 里没有提供示例，只给任务说明。

对比：

- Zero-shot: 直接说“写一篇关于 black holes 的文章”。
- One-shot: 给一个输入-输出示例，再让模型处理新输入。
- Few-shot: 给多个输入-输出示例。

这节重点不是 prompt taxonomy，而是：direct generation 是一次性生成，没有后续自检或改进步骤。

## Where Reflection Helps

课程举了几个适合 reflection 的例子：

- Structured output: HTML table、复杂 JSON、嵌套结构等，可能有格式错误。
- Instructions: 让模型生成步骤说明时，可能漏掉步骤或不连贯。
- Domain names: 生成的名字可能难发音，或者在英文/其他语言里有负面含义。
- Email: 检查 tone、facts、dates、promises 是否准确，再生成下一版。

这些任务的共同点是：有一些可以明确检查的 criteria。

## Reflection Prompt Tips

写 reflection prompt 时，最好包含：

1. 明确要求模型 review / reflect on the first draft。
2. 给出具体检查 criteria。
3. 要求模型基于发现的问题生成 revised draft。
4. 如果有事实、日期、承诺、上下文，确保这些信息在模型 context 中。

例子：

```text
Review the first draft.
Check tone, factual accuracy, dates, and promises.
If you find problems, write an improved next draft.
```

更具体的 criteria 会让 reflection 更有方向。

## Design Takeaways

- 不要把 reflection 当成默认必须加的步骤；先看 direct generation baseline。
- Reflection 适合那些“第一次生成容易漏、错、格式不稳、语气不准”的任务。
- Reflection prompt 要写成有目标的 review，而不是泛泛地说“make it better”。
- Criteria 越具体，reflection 越可能有效。
- 读优秀开源项目里的 prompts，是提升 prompt writing 的好方法。
- 反思是否有效应该用 eval 比较，而不是只凭输出看起来更顺眼。

## Failure Modes

- Criteria 太模糊，模型只做表面润色。
- Reflection 改善了语气，但没有修复事实问题。
- 对简单任务增加 reflection，收益很小但成本上升。
- 没有 direct baseline，无法判断 reflection 是否真的提升。
- Reflection prompt 缺少必要上下文，导致模型无法验证 facts。

## Implementation Hooks

- Baseline: 为每个任务先保存 direct generation 输出。
- Criteria checklist: 把 reflection prompt 写成 checklist，例如 format、completeness、tone、facts、pronunciation、negative connotations。
- Eval: 对比 direct vs reflection 的 error rate、human preference、LLM-as-judge 或 task-specific metrics。
- Prompt library: 收集高质量 reflection prompts，沉淀可复用模板。
- Cost tracking: 记录 reflection 带来的额外 tokens、latency 和质量收益。

## My Questions

- 哪些任务应该默认先试 direct generation，哪些任务可以直接上 reflection？
- Reflection prompt 里 criteria 写得多细最合适？
- 如何设计 eval 来证明 reflection 比 direct generation 更好？
- Reflection 改善“表面质量”和改善“事实正确性”之间怎么区分？

## Related

- [Reflection to Improve Outputs of a Task](01-reflection-to-improve-outputs-of-a-task.md)
- [Reflection pattern](../../concepts/reflection-pattern.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 reflection pattern concept card。
- [ ] 下一节 chart generation workflow 后，补充 multimodal reflection 的例子。
- [ ] 后续 lab 时建立 direct vs reflection 对照实验。

