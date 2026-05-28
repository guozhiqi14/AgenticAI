---
course: Agentic AI
module: 1
lesson: 3
title: Degrees of Autonomy
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/zqs9ty/degrees-of-autonomy
watched_at: 2026-05-28
reviewed_at:
tags: [autonomy, agentic-workflows, tool-use, control]
---

# Degrees of Autonomy

## TL;DR

Agentic system 不是简单的“是 agent”或“不是 agent”。它更像一个 autonomy spectrum：有些 workflow 的步骤由工程师固定写死，有些 workflow 会让 LLM 决定更多计划、工具使用和迭代路径。

## Core Ideas

- "Agentic" 作为形容词很有用，因为它让我们讨论自主程度，而不是陷入二元定义之争。
- 关键设计问题是：哪些决策由工程师固定，哪些决策交给 LLM 在运行时做？
- 低自主性的系统依然很有价值。它可以是 deterministic workflow，LLM 主要负责在固定步骤里生成内容。
- 更高自主性的系统会让 LLM 选择 action、tool、信息收集量、是否反思、是否回头补充资料。
- 高度自主的系统甚至可能生成新的 function 或 tool，但控制、调试和评估都会更难。

## Example From The Lesson

这节课用 research/writing task 来解释 autonomy spectrum，比如让系统写一篇关于 black holes 的文章。

在低自主性一端，工程师可以预先定义 workflow：

1. Generate search queries.
2. Call web search.
3. Fetch pages.
4. Write the essay.

在更高自主性一端，LLM 可以决定：

- 要搜索网页、新闻、论文，还是其他来源。
- 要 fetch 多少页面。
- PDF 是否需要转换成文本。
- 在输出前是否需要 revise、reflect，或者继续补充证据。

## Web Search vs. Web Fetch

这节里出现的 web search 和 web fetch 是两个不同粒度的工具：

- Web search: 用关键词去搜索，返回候选网页列表，通常包括标题、摘要、URL、排名等。它回答的是“有哪些可能相关的来源？”
- Web fetch: 给定一个具体 URL，把这个页面或文件的内容取回来，让 LLM 可以阅读、摘要或引用。它回答的是“这个具体来源里到底写了什么？”

一个简单类比：web search 像在图书馆目录里找书，web fetch 像把某一本书拿下来翻开读。

在 agentic workflow 里，这两步分开很重要，因为 LLM 可能先决定搜什么，再决定哪些结果值得 fetch。更高 autonomy 的 agent 甚至可以自己决定 fetch 几个页面、是否继续搜索、是否读取 PDF 或把 PDF 转成文本。

## Diagram Convention

课程使用了一套固定图示语言：

- Red: user input 或 input document。
- Gray: LLM calls。
- Green: software/tool actions，比如 web search、web fetch、API calls 或 code execution。

## Autonomy Spectrum

| Level | Who decides the steps? | Tool choice | Predictability | Typical use |
| --- | --- | --- | --- | --- |
| Less autonomous | Mostly programmer | Hard-coded | Higher | Reliable business workflows |
| Semi-autonomous | Programmer sets boundaries, LLM chooses within them | Predefined tools | Medium | Research, routing, support, analysis |
| Highly autonomous | LLM can choose or create many steps | Flexible or generated tools | Lower | Open-ended tasks, active research |

## Design Takeaways

- 先判断任务需要什么 autonomy level，而不是纠结自己是不是在做“真正的 agent”。
- 更高自主性不自动等于更好。它带来灵活性，也带来不可预测性、评估成本和控制风险。
- 很多有商业价值的产品其实在低自主性一端：workflow 固定，LLM 在特定步骤里做判断或生成。
- Tool use 是 autonomy 的关键杠杆。工具调用写死在代码里，控制权更多在工程师；让 LLM 选择工具，控制权就更多转移给模型。
- 合适的 autonomy level 取决于风险、成本、可逆性、可观测性，以及任务本身的不确定性。

## Failure Modes

- 对需要合规、可重复、成本可控的任务给了过多 autonomy。
- 对开放任务给了太少 autonomy，导致固定 workflow 漏掉重要路径。
- 让 LLM 自己选工具，但没有 logging、guardrails 或 evals。
- 把“更 agentic”和“更有用”混为一谈。
- 做 reflection 或 retry loop，但没有清晰的停止条件。

## Implementation Hooks

- Prompting: 明确告诉模型哪些事情可以决定，哪些事情是固定的。
- Tools: 在允许模型选工具之前，先给一个有边界的 tool set。
- Search/Fetch: 把“发现候选来源”和“读取具体来源”拆成两个工具，有助于控制成本、记录决策，并评估 agent 是否选对了信息来源。
- Workflow: 高风险步骤用 deterministic orchestration，模糊步骤再给 LLM 选择权。
- Evals: 不只评估 final answer，也评估 action selection 和 stopping behavior。
- Human-in-the-loop: 对昂贵、不可逆、敏感的工具调用加入人工确认。
- Logging: 记录模型选择过哪些步骤，方便失败后复盘。

## My Questions

- 在开始实现前，我应该如何选择 autonomy level？
- 真实产品里，哪些 decision 应该交给 LLM，哪些应该留在代码里？
- 什么 eval 能抓到 bad action choice，而不只是 bad final text？
- reflection 什么时候值得额外 latency 和 cost？

## Related

- [Autonomy spectrum](../../concepts/autonomy-spectrum.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 autonomy concept card。
- [ ] Module 1 完成后补复习题。
- [ ] 学完 tool-use module 后回看这篇。
