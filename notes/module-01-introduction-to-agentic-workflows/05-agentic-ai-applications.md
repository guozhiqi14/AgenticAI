---
course: Agentic AI
module: 1
lesson: 5
title: Agentic AI Applications
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/bvuwkr/agentic-ai-applications
watched_at: 2026-05-29
reviewed_at:
tags: [applications, task-decomposition, computer-use, human-in-the-loop, workflow-design]
---

# Agentic AI Applications

## TL;DR

Agentic AI 更容易落地在“流程清晰、步骤可拆、主要处理文本、有明确工具/API 可调用”的任务上。越需要动态规划、浏览器操作、多模态理解、不可预测页面交互，系统就越难做稳定。

## Core Ideas

- 很多业务场景天然适合 agentic workflow，因为它们本来就有标准作业流程。
- 容易实现的场景通常有清晰的 step-by-step process，例如发票处理、基础订单查询。
- 更难的场景需要 agent 自己决定步骤，例如通用客服问题、退货政策判断、库存查询组合。
- Computer use agents 很有潜力，但当前仍然不够可靠，尤其是网页加载慢、页面结构复杂、视觉或交互理解困难时。
- 设计 agentic workflow 的关键能力之一，是把复杂任务拆成 discrete steps，这也衔接到下一节 task decomposition。

## Examples From The Lesson

### Invoice Processing

输入是一张 invoice PDF，目标是提取关键信息并写入数据库。

典型 workflow:

1. PDF to text，把发票转换成 LLM 能处理的文本或 markdown。
2. 判断文档是否真的是 invoice。
3. 提取 biller、address、amount due、due date 等字段。
4. 调用 database API 保存结构化记录。

这个例子相对容易，因为目标明确、流程固定、输出字段清晰。

### Basic Customer Order Inquiry

输入是客户邮件，目标是根据订单信息起草回复。

典型 workflow:

1. 从邮件里提取 customer name、order details 等信息。
2. 查询 order database。
3. 基于查询结果起草邮件。
4. 把草稿放进 human review queue，由人批准后再发送。

这里的 human-in-the-loop 很重要：agent 负责准备答案，人负责最后审核。

### General Customer Service Agent

如果客户问题更开放，比如问库存、退货政策、是否符合退货条件，workflow 会更难。

难点在于：步骤不一定提前知道。Agent 可能需要自己规划：

- 先查用户是否买过商品。
- 再查购买日期和商品状态。
- 再查退货政策。
- 如果符合条件，再生成 return label，并更新 database 状态。

这类任务比基础订单查询更 agentic，因为它需要 runtime planning。

### Computer Use Agents

Computer use agent 指的是让 agent 操作浏览器或桌面软件，像人一样点击、输入、读页面、调整策略。

课程例子是让 agent 检查航班座位。它先尝试航空公司网页，遇到问题后转到 Google Flights，再回到航空公司页面确认结果。

这个方向很有前景，但目前可靠性仍然不足。网页加载慢、页面结构变化、视觉理解不准确、表单交互复杂，都会让 agent 出错。因此不适合直接用于 mission-critical applications。

## Difficulty Spectrum

| Easier | Harder |
| --- | --- |
| 流程清晰 | 步骤需要运行时规划 |
| 有 SOP 或标准业务流程 | 用户问题开放、分支很多 |
| 输入主要是文本 | 输入包含复杂视觉、音频或多模态内容 |
| 工具有清晰 API | 需要操作浏览器或 GUI |
| 输出格式明确 | 成功标准模糊 |
| 可以 human review | 动作不可逆或高风险 |

## Design Takeaways

- 选 agentic AI 应用时，先找“已有 SOP 的流程”，而不是一上来做最开放的助手。
- 文本输入、明确 schema、清晰 API、可审核输出，是早期落地的友好条件。
- 如果任务步骤不固定，就需要 planning、tool selection、state tracking 和更强 eval。
- Computer use 很诱人，但它把不稳定性从 API 世界带到了网页和 GUI 世界，可靠性难度更高。
- Human review queue 是把 agent 用进业务流程的实用方式：先让 agent draft 或 prepare，再让人 approve。

## Failure Modes

- 把没有明确流程的开放任务当作简单 extraction task 来做。
- 让 agent 直接对外发邮件、改数据库或执行高风险动作，缺少 review/approval。
- 低估 PDF、网页、表格、图片等输入格式带来的 parsing 难度。
- 依赖 browser automation 做关键业务，但没有应对页面变化、加载失败、表单异常。
- 没有把任务拆成可观察的步骤，导致失败时不知道是 extraction、planning、tool call 还是 final response 出错。

## Implementation Hooks

- Application selection: 优先选有 SOP、文本输入、明确输出字段、可人工审核的任务。
- Task decomposition: 把业务流程拆成 extraction、classification、lookup、draft、approval、write-back 等步骤。
- Tools: 能用 API 就优先用 API；browser/computer use 留给没有 API 或必须操作 GUI 的场景。
- Human-in-the-loop: 对发邮件、付款、退货、数据库更新等动作加入审核或确认。
- Evals: 分别评估字段提取、意图判断、工具选择、数据库查询、最终回复质量。
- Reliability: 对页面加载、工具失败、信息缺失、用户输入模糊建立 fallback。

## My Questions

- 对一个业务流程，如何判断它是否已经足够 SOP 化，适合做 agentic workflow？
- 什么情况下应该用 browser/computer use，而不是要求系统提供 API？
- Human review 应该放在哪些步骤：最终输出前，还是每个高风险工具调用前？
- 对通用客服这种开放场景，如何设计 eval 覆盖足够多的分支？

## Related

- [Degrees of Autonomy](03-degrees-of-autonomy.md)
- [Benefits of Agentic AI](04-benefits-of-agentic-ai.md)
- [Agentic application fit](../../concepts/agentic-application-fit.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 application-fit concept card。
- [x] 下一节 `Task decomposition` 后，回头补充这里的 workflow 拆解方法。
- [ ] 后续 lab 可以选 invoice processing 或 order inquiry 做低风险原型。
