---
course: Agentic AI
module: 5
lesson: 2
title: "Creating and executing LLM plans"
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/wr78er/creating-and-executing-llm-plans
watched_at: 2026-07-14
reviewed_at:
tags: [planning, structured-output, json, xml, plan-schema, executor]
---

# Creating and Executing LLM Plans

## TL;DR

上一节讲 planning workflow 的大图：LLM 先生成 plan，再逐步执行。这节补上一个关键工程细节：plan 不应该只是自然语言 checklist，而应该尽量是 downstream code 能解析的结构化格式，例如 JSON 或 XML。这样 executor 才能清楚知道每一步的 description、tool name、arguments 和 step order，从而稳定地一步一步执行 plan。

## Source Handling

本笔记基于课程页 `Show Transcript` 面板整理；本次读取到 29 条去重 transcript cues。Transcript 只用于理解和总结，不保存完整逐字稿。

## Core Ideas

- 要让 LLM 生成可执行 plan，prompt 里需要明确告诉模型：可用工具有哪些，以及 plan 应该用什么结构输出。
- 许多开发者会要求 LLM 用 JSON 输出 plan，因为 downstream code 很容易 parse JSON。
- 结构化 plan 比纯英文描述更适合执行：程序可以明确读出 step list、step number、description、tool 和 arguments。
- XML 也可以表达清晰边界；Markdown 比 JSON/XML 更容易产生解析歧义。
- Plain text 最不可靠，因为它给人读起来可以，但给程序解析会有很多模糊空间。
- 下一节会引出更进一步的方法：让 LLM 用 code 表达 plan。

## Why Structure Matters

如果 plan 只是自然语言：

```text
先查商品描述，再查库存，最后查价格。
```

人能理解，但 executor 很难稳定判断：

- 一共有几步？
- 每一步该调用哪个 tool？
- tool arguments 是什么？
- 第一步结果如何传给第二步？
- 如果某一步失败，应该重试哪一步？

如果 plan 是 JSON，就可以把这些内容变成明确字段：

```json
{
  "steps": [
    {
      "step": 1,
      "description": "Find sunglasses that match the requested shape.",
      "tool": "get_item_descriptions",
      "arguments": {
        "query": "round sunglasses"
      }
    },
    {
      "step": 2,
      "description": "Check inventory for the matching products.",
      "tool": "check_inventory",
      "arguments": {
        "product_ids_from_step": 1
      }
    }
  ]
}
```

这不是课程原文，而是我按这节精神写的简化示例。关键点是：plan schema 让下一层代码知道如何执行，而不是靠正则硬猜一句英文的含义。

## Prompt Shape

课程给出的 prompt 思路可以抽象成：

```text
You have access to the following tools:
- tool A: ...
- tool B: ...
- tool C: ...

Create a step-by-step plan in JSON format.
The JSON should include:
- step number
- description
- tool to use
- arguments for the tool
```

这和 tool schema 很像：你不是只告诉模型“帮我计划一下”，而是给它一个输出 contract。模型生成的 plan 必须符合这个 contract，后续代码才能执行。

## JSON, XML, Markdown, Plain Text

| Format | Strength | Risk |
| --- | --- | --- |
| JSON | 程序最容易 parse；适合传给 executor | 模型可能输出 invalid JSON，需要 validation |
| XML | tag 边界清晰；适合嵌套文本 | parser 和 prompt 设计稍复杂 |
| Markdown | 人类可读性好 | 对程序来说边界不够稳定 |
| Plain text | 最自然，最省 prompt | 解析歧义最大，不适合稳定执行 |

实战里，JSON 和 XML 更适合作为可执行 plan 的载体。Markdown 和 plain text 可以用于展示给人看，但不应该作为高可靠 executor 的唯一输入。

## Executor Mental Model

这节隐含了一个很重要的软件工程结构：

```text
planner LLM
  -> structured plan
  -> parser / validator
  -> executor loop
  -> tool call(s)
  -> step result(s)
  -> final response
```

这里的 parser/validator 很关键。它像一个“入口检查员”：

- 先确认 JSON/XML 是否格式合法。
- 再确认每一步字段是否完整。
- 再确认 tool name 是否在 allowlist 里。
- 再确认 arguments 是否符合 tool schema。
- 再把合法 step 交给 executor。

对你熟悉的 Python/SQL 视角，可以把 structured plan 理解成一张 workflow table：每一行是一个 step，每一列是 `step_id`、`description`、`tool_name`、`arguments`、`depends_on`、`status`。Executor 就是按这些行去执行。

## Design Takeaways

- Planning 的第一步是让模型会“想步骤”；第二步是让这些步骤变成机器能执行的结构。
- Plan schema 是 planning workflow 的 API contract。
- JSON/XML plan 可以降低解析歧义，让 downstream workflow 更系统地执行。
- 结构化输出不等于可靠执行；仍然需要 validation、tool allowlist 和 error handling。
- 如果 plan 是给人看的，可以保留自然语言；如果 plan 是给程序执行的，要优先结构化。
- Plan 的字段设计会直接影响 executor 能否记录 trace、做 eval 和进行失败重试。

## Failure Modes

- 模型输出 invalid JSON，parser 直接失败。
- JSON 字段存在，但 tool name 不在允许列表里。
- arguments 结构不符合 tool schema。
- step description 很清楚，但 tool/arguments 太含糊，executor 仍然无法执行。
- Markdown 或 plain text 看起来漂亮，但程序解析不稳定。
- Plan schema 太宽松，导致模型输出自由文本塞进结构化字段里。
- 没有 validation，错误 plan 被直接执行。

## Implementation Hooks

- 用 JSON schema 或 Pydantic model 定义 plan schema。
- Planner prompt 中明确给出字段名、类型、示例和禁止项。
- Parser 只接受结构化 plan，不从自由文本里猜工具名。
- Validator 检查 `tool` 是否在 registry / allowlist 中。
- Arguments validator 复用每个 tool 的参数 schema。
- Executor 为每个 step 记录 `status`、`result`、`error`、`latency`、`cost`。
- 对 invalid plan 加一个 repair loop：让 LLM 根据 validation error 只修 plan，不执行工具。
- 对高风险 tool，在执行前插入 human approval。

## Data Analyst Translation

如果做数据分析 agent，plan 可以像这样结构化：

```json
{
  "steps": [
    {
      "step": 1,
      "description": "Define retention metric and date window.",
      "tool": "metric_definition_lookup",
      "arguments": {
        "metric": "new_user_retention",
        "window": "last_3_months"
      }
    },
    {
      "step": 2,
      "description": "Query retention trend by week.",
      "tool": "sql_query",
      "arguments": {
        "query_purpose": "weekly retention trend",
        "depends_on": 1
      }
    }
  ]
}
```

真正生产化时，不一定让 LLM 直接写完整 SQL；可以先让它写 `query_purpose`、维度、时间范围和指标名，再由更受控的 SQL component 生成 SQL。这样既保留 planning 的灵活性，又减少自由 SQL 带来的风险。

## My Notes

- 这节像是把 planning 从“思维过程”变成“系统接口”。只有接口稳定，executor 才能稳定。
- JSON plan 对我来说很像数据 pipeline 的 DAG spec：不是最终结果，而是告诉系统下一步怎么跑。
- 这也解释了为什么 schema 设计重要：plan schema 写得好，后续 eval、trace、debug、重试都会简单很多。
- 课程下一节讲 code-as-plan，可能会把 plan 的表达能力进一步提高，但安全和 sandbox 也会更重要。

## Open Questions

- Plan schema 最小字段集应该是什么，才能兼顾执行、trace、eval 和 human review？
- Planner 输出 invalid JSON 时，应该自动 repair 几次？什么时候停止？
- 对数据分析 agent，planner 能不能直接输出 SQL，还是应该输出受控 query spec？
- JSON plan 和 code-as-plan 的边界在哪里？什么时候从 JSON 升级到 code？
- Plan schema 应该允许条件分支和循环吗？如果允许，怎么避免失控？

## Related

- [Planning workflows](01-planning-workflows.md)
- [Planning pattern](../../concepts/planning-pattern.md)
- [Tool schema](../../concepts/tool-schema.md)
- [Tool use](../../concepts/tool-use.md)
- [Code execution tool](../../concepts/code-execution-tool.md)

## Next Actions

- [x] 更新 Module 5 README / summary。
- [x] 扩展 planning pattern 概念卡。
- [x] 增加 glossary / open questions。
- [ ] 下一节继续看 `Planning with code execution`，重点理解 code-as-plan、sandbox 和 executor 的关系。
