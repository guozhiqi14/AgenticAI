---
course: Agentic AI
module: 4
lesson: 1
title: "Evaluations (evals)"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/pu5xbl/evaluations-(evals)
watched_at: 2026-07-09
reviewed_at:
tags: [evals, end-to-end-eval, objective-eval, llm-as-judge, ground-truth, error-analysis]
---

# Evaluations (evals)

## TL;DR

Agentic workflow 的有效改进路径通常是：先快速做出一个安全、可观察的 prototype，再看真实 output 中哪里失败，把最常见或最重要的失败模式转成 eval。Eval 不必一开始很大，10 到 20 个样例就可以帮助你从“凭感觉调 prompt”进入“用指标跟踪改进”的状态。

## Core Ideas

- 不要试图在动手前预测所有错误；agentic system 往往要跑起来之后，才知道哪里不稳。
- Quick and dirty system 和 quick and dirty eval 都有价值，前提是安全、负责、可观察。
- Eval 应该从真实 failure mode 出发，而不是为了有指标而造指标。
- End-to-end eval 评估整个 workflow 的最终输出，不先拆中间组件。
- Objective eval 用代码判断，适合格式、字数、日期、schema 等明确标准。
- LLM-as-a-judge 适合判断语义覆盖、质量、清晰度等不适合简单 pattern matching 的标准。
- Eval 可以有 per-example ground truth，也可以没有；两者都常见。
- Eval 本身也要迭代：如果指标和人的判断不一致，就要扩大 eval set 或修改评估方法。

## Lecture Map

1. 先说明 agentic workflow 开发很难提前知道哪里会失败，所以建议先做 quick prototype。
2. 用 invoice processing 说明如何从人工看 output 发现 due date 抽取错误。
3. 把 due date 错误转成小 eval set：人工标注 ground truth，然后用代码比较。
4. 用 marketing caption 说明没有 per-example ground truth 的 objective eval：统一检查输出是否超过 10 words。
5. 用 research agent 说明 LLM-as-a-judge：判断文章是否覆盖 gold standard talking points。
6. 总结 eval 的两条轴：objective vs LLM-as-a-judge；with vs without per-example ground truth。
7. 强调 eval 是迭代产物，最终目标是让 metric 更贴近 human/expert judgment。

## Three Examples

### 1. Invoice Due Date Extraction

任务：从 invoice 中抽取四个 required fields，并保存到 database record。

观察 output 后发现系统经常把 invoice issue date 和 due date 混淆。于是最值得优先做的 eval 不是“整体发票处理质量”，而是 due date extraction accuracy。

可执行做法：

- 找 10 到 20 张代表性 invoice。
- 人工写下每张 invoice 的正确 due date。
- 要求 LLM 用统一格式输出日期，例如 `YYYY-MM-DD`。
- 用 regex 或 parser 抽取模型输出里的日期。
- 用代码比较 extracted date 和 ground truth date。

这个例子属于：

| Axis | Choice |
| --- | --- |
| Evaluation method | Objective code eval |
| Ground truth | Per-example ground truth |
| Metric | Due date accuracy |

### 2. Marketing Caption Length

任务：给商品图片写 Instagram caption，业务要求 caption 最多 10 words。

人工检查后发现 caption 内容大体还可以，但经常超过长度限制。这里不需要给每个样例写一个不同的正确答案，因为每个样例共享同一个约束：word count <= 10。

可执行做法：

- 准备一小组商品图片和用户请求。
- 跑生成 workflow。
- 用代码统计每条 caption 的 word count。
- 统计符合长度要求的比例。

这个例子属于：

| Axis | Choice |
| --- | --- |
| Evaluation method | Objective code eval |
| Ground truth | No per-example ground truth |
| Metric | Length compliance rate |

### 3. Research Agent Key Points

任务：让 research agent 写主题文章，例如黑洞科学突破、租房 vs 买房、机器人采摘水果。

人工检查后发现它有时会漏掉专家认为很重要的观点。这个问题不能靠简单关键词匹配，因为同一个观点可以用很多不同说法表达。

可执行做法：

- 为每个 research prompt 写 3 到 5 个 gold standard talking points。
- 让 agent 生成 essay。
- 用 LLM-as-a-judge 判断 essay 覆盖了几个 key points。
- 要求 judge 返回结构化 JSON，比如 `score` 和 `explanation`。

这个例子属于：

| Axis | Choice |
| --- | --- |
| Evaluation method | LLM-as-a-judge |
| Ground truth | Per-example ground truth |
| Metric | Covered key points count |

## Eval Design Matrix

| | With per-example ground truth | Without per-example ground truth |
| --- | --- | --- |
| Objective code eval | Invoice due date 是否等于人工标注日期 | Caption 是否不超过统一 word limit |
| LLM-as-a-judge | Research essay 是否覆盖每个 topic 的 gold standard points | Chart/report 是否满足同一套 rubric |

这张矩阵是这节最实用的框架。设计 eval 时可以先问两个问题：

1. 这个质量标准能不能用代码稳定判断？
2. 每个样例是否需要自己的 ground truth？

## Design Takeaways

- 先看输出，再决定评什么；不要一开始就假设最重要的问题是什么。
- 小 eval set 是合理起点。它不能代表完整质量，但能让开发进入可比较状态。
- 如果能用代码判断，就优先用 objective eval；更稳定、便宜、可重复。
- 如果标准需要语义理解，再用 LLM-as-a-judge，并强制结构化输出和解释。
- 对数据分析 agent，第一批 eval 应该来自真实错误：SQL 结果错、指标口径漏、图表不清楚、结论缺 evidence。
- Eval 的目标不是替代人工判断，而是逐步把人工判断沉淀成可重复的反馈系统。

## Failure Modes

- 还没看真实 outputs，就先做一套很重的 eval framework，最后评错重点。
- Eval set 太小且长期不扩展，导致系统只对少量样例过拟合。
- 指标容易被优化，但不代表用户真的满意。
- LLM-as-a-judge prompt 太模糊，导致 judge 分数不稳定。
- Per-example ground truth 标注不一致，模型改进被噪声淹没。
- 只看 end-to-end metric，不看 traces，后续仍然不知道应该改哪个组件。

## Implementation Hooks

- Prompting: 对 objective eval，要求模型输出稳定格式，比如日期统一为 `YYYY-MM-DD`、JSON 使用固定字段。
- Tools: 用 regex、schema validator、word count、unit tests 或 SQL execution 构建便宜的 objective checks。
- Workflow: 先跑 end-to-end eval，再根据错误模式进入 component-level eval。
- Memory/state: 保存输入、输出、trace、版本号、prompt/model 配置，方便对比不同版本。
- Evals: 从 10 到 20 个样例开始，记录 metric、人工观察和失败标签。
- Human-in-the-loop: 定期人工抽查 eval 结果，确保 metric 仍然贴近 expert judgment。

## My Notes

- 这节和我们之前的 `Loop Engineering` 很像：先跑一个可观察循环，再用错误分析决定下一步，而不是在白板上幻想完美架构。
- 对我的数据分析场景，最自然的 objective eval 可能是：SQL 执行是否成功、结果是否等于 ground truth、图表字段是否齐全、指标口径是否引用正确。
- LLM-as-a-judge 更适合评“分析有没有洞察”“解释是否覆盖关键因素”“结论是否有 evidence”，但 judge 本身也要被校准。
- 小 eval set 的意义不是“一锤定音”，而是让每次 prompt/tool/workflow 修改都留下可比较证据。

## Open Questions

- 我的数据分析 agent 第一批 20 个 eval examples 应该从哪些真实任务抽样？
- 对业务分析报告，哪些维度能用 objective eval，哪些必须用 LLM-as-a-judge？
- 如果 human judgment 和 eval metric 冲突，应该优先改 eval set、judge prompt，还是 workflow 本身？
- 多久应该扩充一次 eval set，避免只优化旧样例？

## Related

- [Agentic evals](../../concepts/agentic-evals.md)
- [Evaluating Agentic AI (Module 1)](../module-01-introduction-to-agentic-workflows/07-evaluating-agentic-ai-evals.md)
- [Evaluating the Impact of Reflection](../module-02-reflection-design-pattern/05-evaluating-the-impact-of-reflection.md)
- [Module 4 summary](../../review/module-04-summary.md)

## Next Actions

- [x] 新建 Module 4 README。
- [x] 扩展 `concepts/agentic-evals.md`。
- [x] 增加 glossary / open questions。
- [ ] 下一节继续整理 error analysis 和 prioritizing next steps。
