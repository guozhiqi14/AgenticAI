---
course: Agentic AI
module: 4
lesson: 2
title: "Error analysis and prioritizing next steps"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/2ftglp/error-analysis-and-prioritizing-next-steps
watched_at: 2026-07-09
reviewed_at:
tags: [error-analysis, traces, spans, prioritization, component-level-eval, research-agent]
---

# Error Analysis and Prioritizing Next Steps

## TL;DR

Agentic workflow 出错时，不要凭直觉随便挑一个组件改。更高效的做法是读 traces，把最终失败归因到具体 component，再用一个简单 spreadsheet 统计不同组件的错误频率；最后结合“这个组件错得多不多”和“我有没有办法改好它”来决定下一步优先级。

## Core Ideas

- 复杂 agentic workflow 里，同一个最终错误可能由多个组件造成。
- Error analysis 的目标是决定最值得先改哪里，而不是只证明系统有错。
- Trace 是一次运行中所有 intermediate outputs 的集合；span 是某一个步骤的 output。
- 不要只看满意的 case；error analysis 应该聚焦 final output 不满意的 cases。
- 判断某个 component 是否有错，要看它在给定输入下是否明显低于 human expert baseline。
- 上游组件给了坏输入时，下游组件即使产出不好，也不一定应该背锅。
- 系统化 error analysis 可以用 spreadsheet 记录每个 failed case 的组件级失败标签。
- 优先级来自两件事：组件贡献了多少错误，以及你是否有现实可行的改进方案。

## Lecture Map

1. 提出问题：quick prototype 不够好时，应该先改哪个 component？
2. 用 research agent 漏掉 key points 的例子说明，错误可能来自很多步骤。
3. 展开可能原因：search terms 不好、search engine 不好、source selection 不好、fetched documents 没被充分利用。
4. 引入 trace/span：读中间输出，判断哪个步骤低于 human expert 水平。
5. 说明 error analysis 应该聚焦失败样例，而不是平均看所有样例。
6. 用 spreadsheet 记录不同 failed prompts 的组件错误，并统计错误频率。
7. 用错误频率和可改进性决定下一步：例如 search results 45% 不满意，就优先看 search engine/parameters。

## Research Agent Example

最终失败：research agent 写文章时漏掉人类专家会提到的重要观点。

可能原因包括：

| Component | Possible Failure |
| --- | --- |
| Search-term generation | LLM 生成的 search terms 太泛、方向错、没有覆盖关键概念 |
| Web search | Search engine 返回太多博客/大众媒体，缺少高质量 scientific sources |
| Source selection | LLM 没有从 search results 中挑出最值得下载的来源 |
| Web fetch | 页面抓取失败、内容不完整、格式噪声太多 |
| Reading/synthesis | LLM 拿到好文档后仍然忽略关键观点 |
| Final writing | 结构或表达不好，导致关键点没有进入 final answer |

这节的重点是：最终 output 漏点，不代表一定要改 final writing prompt。也可能真正的问题在 search results 阶段，模型从一开始就没有拿到高质量资料。

## Trace And Span

- Trace: 一次 agent run 的完整中间记录，包括每一步 input/output、tool call、source list、选择理由、错误、耗时等。
- Span: trace 里的单个步骤输出，例如 search terms、search results、selected sources、fetched page summary。

可以把 trace 想成一次数据分析 pipeline 的完整日志，span 就是其中某个 node 的输出表。最终报告错了时，不应该只改最后的报告模板，而要沿着 pipeline 往前看每个 node 的中间产物。

## Spreadsheet Error Analysis

一个轻量但有效的做法：

| Prompt / Case | Search terms | Search results | Source selection | Reading/synthesis | Notes |
| --- | --- | --- | --- | --- | --- |
| Recent black hole science | OK | Bad: too many popular articles | Not counted as bad if inputs were poor | Maybe OK | First priority may be search engine |
| Renting vs buying in Seattle | OK | Missing key source | Maybe affected | Maybe affected | Need inspect search query/source quality |
| Robotics for harvesting fruits | Bad: too generic | Bad | Affected | Unknown | Upstream query issue may explain downstream failures |

关键判断：

- 如果 source selection 的输入本来全是低质量来源，就不要轻易把错误归咎于 source selection。
- 如果 search terms 看起来接近 human expert 会写的 query，但 results 很差，就可能应该换 search engine、调 search parameters 或加 domain constraints。
- 如果 search results 很好，但 final essay 还是漏掉 key points，就应该看 reading/synthesis 或 final writing。

## Prioritization Rule

优先改的 component 通常满足两个条件：

1. 它在失败样例中贡献了较多错误。
2. 你有比较明确、成本可控的改进想法。

如果某个 component 错误很多，但完全不知道怎么改，它未必是当前第一优先级。相反，一个错误频率中等但很好修的 component，可能值得先处理。

## Design Takeaways

- Error analysis 是 agentic workflow 开发效率的放大器：它帮你少走“改了几周但整体分数不动”的弯路。
- 先读 trace，再决定改 prompt、tool、retrieval、model、parser 还是 workflow structure。
- 评估 component 时要看 input quality；下游被坏输入拖累时，不能只看它的 output 不好。
- Spreadsheet 是足够好的第一版 error analysis 工具，不需要一开始上复杂平台。
- 对数据分析 agent，trace 至少应该记录：用户问题、schema lookup、SQL draft、SQL execution、result table、chart spec、final explanation。

## Failure Modes

- 只凭直觉改最显眼的组件，例如 final prompt，而真正瓶颈在 retrieval/search。
- 把上游坏输入导致的问题错误归因到下游组件。
- 只看成功 case，导致看不到真正阻碍上线的错误模式。
- Trace 不完整，无法复盘每一步到底发生了什么。
- 只统计错误频率，不考虑是否有可行改进方案。
- 过早自动化 error analysis，反而还没建立人类专家的判断标准。

## Implementation Hooks

- Prompting: 让每个关键步骤输出可读、可审计的 intermediate artifact，例如 search terms、selected sources、reasoning summary。
- Tools: 保存 tool inputs/outputs，特别是 search query、returned URLs、fetched content metadata、SQL execution result。
- Workflow: 每个 component 设计清晰输入输出，方便判断它是否低于 human baseline。
- Memory/state: 给每次 run 分配 ID，把 trace、model version、prompt version、tool version 绑定。
- Evals: 先做 failed-case spreadsheet；等错误标签稳定后，再转成 component-level eval。
- Human-in-the-loop: 让懂业务的人标注哪些 component 输出低于专家水平。

## My Notes

- 这节对我的数据分析 agent 很重要：如果最终洞察错了，原因可能是 schema 没找对、SQL 写错、指标口径错、聚合粒度错、图表选择错，或者 final explanation 过度解读。
- “不要让下游背上游的锅”这个判断很关键。比如 SQL execution 失败可能是 schema tool 给错表，也可能是 SQL generator 自己写错。
- Spreadsheet error analysis 很适合我当前阶段：不用先做复杂 eval 平台，先手动标 20 个失败 case 就能知道哪里最值得改。

## Open Questions

- 数据分析 agent 的 trace/span 应该设计成什么字段，才能支持后续 error analysis？
- 如果一个错误跨多个组件传播，如何给组件分配责任而不重复计数？
- 对内部业务场景，谁来充当 human expert baseline：数据分析师、产品经理，还是指标 owner？
- 什么时候应该把 spreadsheet error analysis 升级成自动化 dashboard？

## Related

- [Evaluations (evals)](01-evaluations-evals.md)
- [Error analysis concept card](../../concepts/error-analysis.md)
- [Agentic evals](../../concepts/agentic-evals.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 `error-analysis` concept card。
- [x] 更新 Module 4 README / summary。
- [ ] 下一节继续看更多 error analysis examples。
