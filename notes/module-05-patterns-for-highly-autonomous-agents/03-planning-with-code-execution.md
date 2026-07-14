---
course: Agentic AI
module: 5
lesson: 3
title: "Planning with code execution"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/egr0a8/planning-with-code-execution
watched_at: 2026-07-14
reviewed_at:
tags: [planning, code-execution, pandas, code-as-plan, sandbox, data-analysis]
---

# Planning With Code Execution

## TL;DR

Planning with code execution 的核心是：不一定让 LLM 输出 JSON plan 再由 executor 逐步翻译执行；如果任务天然可以用代码完成，可以让 LLM 直接写代码，把 plan 表达在代码里，再在安全环境中执行。对数据表、CSV、pandas 分析这类任务，这通常比不断增加微型工具更灵活。但代价也明显：code execution 必须有 sandbox、权限限制、timeout、trace 和错误处理，否则 runtime 行为更难控制。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理；本次读取到 74 条去重 transcript cues。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- Code execution 可以把 plan 变成可执行程序，而不是只输出 JSON/XML 结构。
- 如果给 LLM 一堆固定小工具，复杂数据问题会很快遇到工具覆盖不足和 edge cases。
- Python/pandas 这类通用库已经内置大量函数，LLM 也见过很多调用模式，因此适合让模型组合这些函数完成复杂分析。
- Code-as-plan 不是“没有计划”；计划藏在代码的步骤、函数调用和注释里。
- 当任务可以自然表达成代码时，code execution 可以让 LLM 表达更复杂、更灵活的 plan。
- 课程强调 caveat：执行 LLM 生成的代码需要安全执行环境，最好用 sandbox。
- Planning 提高能力范围，但牺牲可控性；agentic coding 是目前最成熟的 planning 应用之一。

## The Problem With Too Many Tiny Tools

课程用一个销售 spreadsheet 问答场景说明问题。

假设你给 LLM 一组很细的工具：

- get column max
- get column mean
- filter rows
- get column min
- get column median
- sum rows

这些工具能回答一些简单问题，但一旦用户问更复杂的问题，例如按月份比较某类商品销售额、统计上周唯一交易数、查看最近几笔交易金额，系统就会变得很笨重。

你会不断补新工具：

- get unique entries
- get latest rows
- group by month
- parse date
- filter date window
- count distinct

这类做法的问题是：

- 工具数量越来越多，LLM 选择工具更难。
- 每个新问题都可能暴露一个缺失工具。
- 多工具串联步骤很长，latency 和 failure surface 增加。
- 系统维护成本越来越高。

这对数据分析 agent 很熟悉：如果每个 pandas/SQL 操作都封成一个 tool，很快会变成一堆难维护的函数清单。

## Code-As-Plan

更灵活的做法是：让 LLM 写代码来解决当前问题。

```text
user asks a data question
  -> LLM writes Python/pandas code
  -> sandbox executes code
  -> stdout/result/error returns
  -> LLM explains answer or revises code
```

这时 plan 不是单独的 JSON list，而是由代码表达：

```python
import pandas as pd

df = pd.read_csv("sales.csv")
df["date"] = pd.to_datetime(df["date"])

recent = df.sort_values("date").tail(5)
answer = recent[["date", "amount"]]
print(answer.to_string(index=False))
```

这段代码里的 plan 是：

1. 读取数据。
2. 解析日期。
3. 按日期排序。
4. 取最近几行。
5. 输出需要的列。

也就是说，代码既是 plan，也是 executor 可以直接运行的 artifact。

## Why Code Can Be More Expressive

JSON plan 适合表达有限步骤和有限工具调用；代码更适合表达复杂数据处理，因为它天然支持：

- 条件判断。
- 循环。
- 函数组合。
- 变量和中间结果。
- 大量成熟 library function。
- 错误堆栈和调试反馈。

对数据分析来说，pandas 就像一个大型工具箱。与其自己封装几十个小工具，不如在受控 sandbox 中让 LLM 调用 pandas 的现有能力。

这也是为什么课程说 code execution 可以让 LLM 从“有限工具集合”扩展到“一个大型编程语言/library 生态”的原因。

## When To Use Code Execution Planning

适合：

- 数据表、CSV、Excel、日志、JSON 等结构化数据分析。
- 需要组合很多小操作的问题。
- 用户问题变化大，很难提前枚举所有工具。
- 需要快速计算、聚合、过滤、排序、去重或画图。
- 错误可以通过 traceback 自动反馈给 LLM 修正。

不适合：

- 高风险外部动作，例如转账、退款、删数据、发邮件。
- 必须严格合规、每一步都要固定审批的流程。
- 代码执行环境无法安全隔离时。
- 任务主要依赖业务 API，而不是通用计算/数据处理。
- 输出需要强一致事务保证时。

## Safety Boundary

这节和 Module 3 的 code execution 连起来：代码执行能力很强，但 prompt 不是安全边界。

最少需要：

- sandbox：隔离文件系统、环境变量、网络和系统权限。
- timeout：限制无限循环或长时间运行。
- resource limit：限制 CPU、内存、磁盘。
- file allowlist：只允许访问任务需要的数据文件。
- network policy：默认禁网，除非任务明确需要。
- write policy：限制写文件、删文件和外部副作用。
- package policy：控制可 import 的库。
- audit log：记录 code、stdout、stderr、artifacts、latency 和 cost。

如果代码会触发外部世界动作，应该加 human approval。

## Research / Practice Signal

课程提到一个趋势：在一些研究和实践中，让 LLM 通过 code 表达 action/plan，往往比先写 JSON 再翻译成 action 更强；JSON plan 又通常比 plain text plan 更可靠。

我的理解：

```text
plain text plan < JSON/XML plan < code-as-plan
```

但这不是说 code-as-plan 永远最好。它只在任务适合用代码表达、并且有安全执行环境时成立。

## Data Analyst Translation

对你来说，这节非常实用。

如果做数据分析 agent，不建议把所有 pandas 能力都封成几十个工具。更好的路线可能是：

1. 固定少数高价值工具：读取数据、查 schema、查指标口径、执行受控 SQL、运行 Python。
2. 让 LLM 在 Python sandbox 中写 pandas 代码完成临时分析。
3. 把执行结果、traceback 和中间表摘要返回给 LLM。
4. 让 LLM 基于结果写解释、限制条件和下一步建议。
5. 用 eval 检查代码执行是否正确、是否越权、是否解释过度。

这和你的 Python/pandas 背景很贴：与其把每个分析操作做成 API，不如让 agent 写小脚本，并把运行环境和权限管住。

## Design Takeaways

- Code execution 是 planning 的一种高表达力形式：plan 直接写在代码里。
- 工具太细会导致 tool explosion；code execution 可以减少工具膨胀。
- 代码适合表达数据处理、计算和文件转换，不适合无边界地执行外部动作。
- Code-as-plan 的可靠性来自成熟编程语言/library，也来自 execution feedback。
- 安全边界必须在 sandbox 和 runtime 层实现，不能只靠 prompt。
- 对 agentic coding，planning/checklist/code execution 已经是很成熟的组合；其他领域仍在探索。

## Failure Modes

- 没有 sandbox，LLM 生成代码读写了不该访问的文件。
- 代码运行环境缺少依赖包或输入文件，导致失败。
- LLM 写出能运行但逻辑错误的 pandas 代码。
- stdout 太长，把上下文污染。
- 自动 retry 太多次，latency/cost 失控。
- 代码绕过业务规则，例如直接读取 raw data 而不是使用 metric definition。
- 把所有任务都塞给 code execution，忽略更稳定的专用工具或审批流程。
- 生成代码后没有保存 trace，无法复盘答案来源。

## Implementation Hooks

- Code block delimiter: 明确要求模型把可执行代码放在固定 tag 或 code block 中。
- Sandbox runner: 用隔离环境执行代码，限制文件、网络、资源和时间。
- Input mounting: 只挂载任务需要的 CSV/Excel/JSON。
- Result contract: 规定 stdout、artifact、structured result 的返回格式。
- Error feedback: 把 traceback 摘要返回给 LLM，允许有限次数修正。
- Code review gate: 对高风险代码或外部动作加人工确认。
- Eval: 同时评估 final answer、code correctness、runtime safety 和 cost/latency。
- Logging: 保存 generated code、execution result、stderr、artifact path、模型版本和 prompt 版本。

## My Notes

- 这节对数据分析 agent 特别关键：pandas sandbox 可能比一堆细碎工具更自然。
- 但“让 LLM 写代码”不是放飞自我，真正的产品边界在 sandbox、权限、数据访问和 eval。
- Code-as-plan 也解释了为什么 coding agent 强：软件开发本来就可以用代码和测试反馈表达中间步骤。
- 我后面做 lab 时，可以先做一个最小 CSV 分析 sandbox：输入问题和 CSV，让 agent 写 pandas，执行，然后解释结果。

## Open Questions

- 数据分析 agent 的 Python sandbox 最小权限应该是什么？
- 哪些分析任务应该让 LLM 写 pandas，哪些应该走固定 SQL/query spec？
- 如何 eval 代码逻辑正确性，而不只是 eval 最终自然语言回答？
- Code-as-plan 的 trace 应该怎么展示给人看，才能方便 review？
- 自动修代码最多允许几轮，超过后应该如何 fallback？

## Related

- [Creating and executing LLM plans](02-creating-and-executing-llm-plans.md)
- [Planning pattern](../../concepts/planning-pattern.md)
- [Code execution tool](../../concepts/code-execution-tool.md)
- [Tool use](../../concepts/tool-use.md)
- [External feedback](../../concepts/external-feedback.md)

## Next Actions

- [x] 更新 Module 5 README / summary。
- [x] 扩展 planning pattern / code execution tool 概念卡。
- [x] 增加 glossary / open questions。
- [ ] 下一节进入 multi-agentic workflows，关注从单 agent planning 到多 agent collaboration 的边界。
