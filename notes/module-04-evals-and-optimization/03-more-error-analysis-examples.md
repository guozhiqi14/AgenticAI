---
course: Agentic AI
module: 4
lesson: 3
title: "More error analysis examples"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/0kbds1/more-error-analysis-examples
watched_at: 2026-07-09
reviewed_at:
tags: [error-analysis, component-attribution, invoice-processing, customer-email, component-level-eval]
---

# More Error Analysis Examples

## TL;DR

这节用 invoice processing 和 customer email assistant 两个例子，把上一节的 error analysis 方法讲得更具体：先找 final output 不满意的样例，再判断错误来自哪个 component。关键不是把所有问题都算到最后一步，而是沿着 workflow 找“最早、最主要、最值得修”的失败来源。

## Core Ideas

- Error analysis 要聚焦失败样例，不要把已经做对的样例混进来稀释判断。
- 同一个 final error 可能由多个 component 同时造成，所以错误比例不一定加起来等于 100%。
- 归因时要看中间产物：如果上游给了坏输入，下游表现差不一定是下游自己的错。
- Invoice due date 错误可能来自 PDF-to-text，也可能来自 LLM extraction。
- Customer email 错误可能来自 database query、database data quality，也可能来自 final email drafting。
- Error analysis 的产物应该直接指向下一步：优先改错误最多、且有明确修法的 component。
- 当一个 component 被确定为瓶颈后，下一步通常是为它建立 component-level eval。

## Lecture Map

1. 说明 error analysis 需要看多个例子来训练判断直觉。
2. 案例一：invoice processing 的 due date 错误。
3. 把错误拆成两个可能来源：PDF-to-text 失败，或 LLM 从正确文本中抽错日期。
4. 案例二：customer email assistant 的最终回复不满意。
5. 把错误拆成 database query、database data quality、email drafting 三类。
6. 强调错误类型不一定互斥，比例可能超过 100%。
7. 引出下一节：为了高效改进某个 component，需要做 component-level eval。

## Example 1: Invoice Processing

Workflow:

```text
invoice PDF -> PDF-to-text -> LLM field extraction -> database record
```

已知 final failure：系统经常抽错 invoice due date。

Error analysis 的做法：

- 找 10 到 100 个 due date 抽错的 invoices。
- 对每个失败样例查看 PDF-to-text 的输出。
- 如果 PDF-to-text 已经把日期文字搞坏，甚至人也难以判断 due date，则归因到 PDF-to-text。
- 如果 PDF-to-text 输出足够清楚，但 LLM 仍然抽成 invoice date 或其他日期，则归因到 LLM extraction。
- 统计每类错误出现频率，再决定先优化 OCR/PDF parser，还是优化 LLM extraction prompt / schema / examples。

一个轻量 spreadsheet 可以这样设计：

| Case | PDF-to-text quality | LLM extraction | Likely cause | Next action |
| --- | --- | --- | --- | --- |
| Invoice 01 | Good | Picked invoice date | LLM extraction | Clarify due-date instruction |
| Invoice 02 | Bad OCR around date section | Also wrong | PDF-to-text | Improve parser/OCR |
| Invoice 03 | Good | Missing due date | LLM extraction | Add examples / output validation |

课程里的核心判断：如果统计后发现 LLM extraction 造成更多错误，就不要花几周去调 PDF-to-text，因为那可能对整体表现没什么帮助。

## Example 2: Customer Email Assistant

Workflow:

```text
customer email -> LLM writes database query -> database returns order info -> LLM drafts response -> human review
```

已知 final failure：最终 email response 不满意。

可能原因：

| Component | Failure Pattern |
| --- | --- |
| Query generation | LLM 查错表、用错字段、漏掉 order id、SQL 条件错 |
| Database | 数据本身错误、缺失、过期或不一致 |
| Email drafting | 拿到正确 order info 后，语气、解释、承诺或行动建议写得不好 |

Error analysis 的做法：

- 找一批最终回复不满意的 emails。
- 查看 LLM 生成的 query 是否能取到正确 customer/order information。
- 如果 query 是对的，再检查 database 是否真的有正确数据。
- 如果数据也是对的，再检查 final email drafting 是否低于 human reviewer 的标准。
- 统计哪类错误最常见。

如果 75% 的失败都来自 database query generation，而 email drafting 只有 30% 的失败，那么第一优先级通常是改 query generation；第二优先级才是 final email prompt。

## Non-Mutually Exclusive Errors

错误比例不一定加起来等于 100%，因为一个失败样例可以同时有多个问题。

例如：

- Query 写错了，导致取错数据。
- Database 本身也有脏数据。
- Final email 还写得不清楚。

这种情况下，一个 case 可以同时标注多个 component。关键是后续做 prioritization 时要判断：哪个错误最常见、最早发生、最可能带来整体改进。

## Design Takeaways

- Error analysis 的第一版可以很朴素：一个 spreadsheet 就够。
- 先聚焦失败样例，能更快找到最值得修的 component。
- 不要因为某个 component 看起来“技术上更底层”就默认先修；先看它是否真是主要错误来源。
- 对数据分析 agent，类似拆法是：自然语言理解、schema lookup、SQL generation、数据库数据质量、结果解释、图表生成、最终回复。
- 如果发现 SQL generation 是主要问题，下一步就应该做 SQL generation component-level eval，而不是继续只看 end-to-end answer。

## Failure Modes

- 混入大量成功样例，导致错误归因不够集中。
- 看到 final email 不好，就只改 email drafting prompt，但真正原因是 query generation。
- 看到 invoice 日期错，就先调 PDF parser，但实际 PDF-to-text 已经足够好。
- 错误标签设计得互斥，导致一个 case 的多个真实问题被压成单一原因。
- 统计了错误频率，但没有把它转成明确的 component-level 改进计划。

## Implementation Hooks

- Prompting: 对 LLM extraction / query generation，要求输出结构化字段和可审计理由，方便 error analysis。
- Tools: 保存 PDF-to-text output、SQL/query text、database response、final email draft。
- Workflow: 每个 component 都要有可查看的 intermediate artifact。
- Memory/state: 给失败样例打标签，例如 `pdf_text_bad`、`llm_extraction_wrong`、`query_wrong_table`、`email_tone_bad`。
- Evals: 从 error tags 中挑最高频 component，建立 component-level eval。
- Human-in-the-loop: 让 reviewer 不只改最终回复，也标注错误来自哪个 component。

## My Notes

- 这节很像数据分析中的 root-cause analysis：指标坏了不要只看最终报表，要看数据采集、ETL、口径、SQL、可视化、解释链路。
- 对我的场景，customer email 的 query generation 很像数据分析 agent 的 SQL generation。最终答案错时，第一步应该看 SQL 是否查对表/字段/条件，而不是直接改回答语气。
- “错误不互斥”很重要。实际业务里常常是 query 有问题、数据也有问题、解释还过度自信；记录多个标签比强行单选更真实。

## Open Questions

- 数据分析 agent 的 error tags 应该允许多选到什么粒度？
- 如果一个失败样例多个 component 都有错，如何判断哪个 component 是最高优先级？
- 对 SQL generation，component-level eval 应该评 SQL 文本本身，还是只评执行结果？
- Database/data quality 问题应该进入 agent eval，还是单独进入数据治理流程？

## Related

- [Error analysis and prioritizing next steps](02-error-analysis-and-prioritizing-next-steps.md)
- [Error analysis concept card](../../concepts/error-analysis.md)
- [Module 4 summary](../../review/module-04-summary.md)

## Next Actions

- [x] 更新 error analysis concept card。
- [x] 更新 Module 4 README / summary。
- [ ] 下一节学习 component-level evaluations。
