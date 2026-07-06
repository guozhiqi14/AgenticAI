---
course: Agentic AI
module: 3
lesson: 1
title: What are tools?
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/3s0czq/what-are-tools%3F
watched_at: 2026-07-06
reviewed_at:
tags: [tool-use, functions, function-calling, agentic-workflows, external-information]
---

# What Are Tools?

## TL;DR

Tool use 是让 LLM 可以请求调用开发者提供的函数。函数可以获取当前信息、查询数据库、执行计算、搜索网页或操作业务系统。重点不是 LLM 自己真的“会执行代码”，而是它判断什么时候需要工具，然后请求系统调用工具，再根据工具返回结果生成最终答案。

## Core Ideas

- LLM 本身只知道训练时学到的信息，不知道当前时间、实时网页、你的数据库状态。
- Tool 就是开发者提供给 LLM 的 function，LLM 可以在需要时请求调用。
- 工具调用的基本流程是：用户问题 -> LLM 判断要不要调用工具 -> 工具执行并返回结果 -> LLM 基于结果回答。
- 同一组工具下，LLM 可以选择不用工具，直接回答。
- 这和 developer hard-code 某一步工具调用不同：tool use 中，是否调用、调用哪个工具，通常由 LLM 决定。
- 一个应用可以给 LLM 多个工具，让它按任务选择和组合。

## Basic Workflow

```text
User prompt
  -> LLM sees available tools
  -> LLM requests a tool call if needed
  -> Tool/function runs outside the LLM
  -> Tool result is added back into conversation/context
  -> LLM generates final answer or decides next tool call
```

用 Python 类比：

```python
def get_current_time():
    ...

def search_web(query):
    ...

def query_database(sql):
    ...
```

这些函数本来是普通 Python 函数。把它们“暴露”给 LLM 后，LLM 就可以在合适的时候请求系统调用它们。

## Example: Current Time

如果用户问“现在几点？”，纯 LLM 不应该假装知道当前时间，因为模型训练完成后并不会自动知道实时信息。

如果开发者提供 `getCurrentTime` 工具，流程会变成：

1. 用户问当前时间。
2. LLM 判断需要 current time。
3. LLM 请求调用 `getCurrentTime`。
4. 函数返回当前时间。
5. LLM 把这个结果组织成自然语言回答。

如果用户问“绿茶含多少咖啡因？”，同样有 `getCurrentTime` 工具，LLM 也可以选择不调用它，因为这个问题不需要当前时间。

## Tool Use vs Hard-Coded Tool Call

| Pattern | Who Decides? | Example |
| --- | --- | --- |
| Hard-coded tool call | Developer | Research agent 每次固定先 web search |
| Tool use | LLM chooses from available tools | LLM 判断这个问题是否需要 `getCurrentTime` 或 `search_web` |

这个区别很重要：hard-coded workflow 更可控；LLM-selected tool use 更灵活，但也更需要 eval、日志和权限控制。

## Examples From The Lesson

### Web Search

用户问“Mountain View 附近有哪些意大利餐厅？”如果 LLM 有 web search tool，它可以生成搜索 query，拿到结果，再总结推荐。

### Database Query

零售店用户问“哪些客户买过白色太阳镜？”如果 LLM 有 database query tool，它可以查询 sales table，再根据查询结果回答。

对你来说，这个例子很贴近数据分析 agent：LLM 负责理解问题和生成查询意图，工具负责真实查询数据库。

### Calculation

用户问“500 美元，年利率 5%，10 年后是多少？”如果有 interest calculation tool，LLM 可以调用它。后面也会看到另一种方式：让 LLM 写代码或数学表达式，再执行得到答案。

### Calendar Assistant

用户说“帮我找周四空档并和 Alice 约会”。系统可以提供多个工具：

- `check_calendar`
- `make_appointment`
- `delete_appointment`

LLM 可能先调用 `check_calendar` 找空档，再选择一个时间，最后调用 `make_appointment` 创建日程。

## Design Takeaways

- Tool use 扩展了 LLM 的能力边界：实时信息、私有数据、计算、外部动作都可以通过工具完成。
- 工具应该围绕应用真实需要设计，不是工具越多越好。
- 工具名称、输入参数、返回结果要清楚，否则 LLM 很难稳定选择和使用。
- 多工具 workflow 里，trace 很重要：要记录 LLM 为什么选工具、传了什么参数、工具返回什么。
- 有副作用的工具，比如发邮件、改日历、下单、写数据库，需要 human review 或权限保护。
- Tool use 是上一模块 external feedback 的自然延伸：工具返回的结果会成为 LLM 的新信息。

## Failure Modes

- LLM 明明不需要工具却调用工具，增加 latency/cost。
- LLM 需要工具却没调用，导致答案凭空猜测。
- 工具描述不清，LLM 选错工具或传错参数。
- 工具返回结果太原始，LLM 难以理解。
- 有副作用的工具缺少确认机制，造成错误操作。
- 没有 trace，无法 debug tool selection 或 tool arguments。

## Implementation Hooks

- Tool design: 从真实任务出发设计工具，例如 search、query database、calculate、create calendar event。
- Tool schema: 工具输入输出要结构化，尽量让参数名清楚。
- Tool routing: 观察 LLM 是否能在需要时调用正确工具。
- Permissions: 对高风险工具加 human-in-the-loop。
- Evals: 评估 tool selection、argument correctness、final answer correctness。
- Logging: 记录 tool calls、arguments、results、latency、errors。

## My Questions

- 数据分析 agent 最核心的第一批 tools 应该是什么：SQL 查询、指标口径检索、表 schema 查询、还是 Python 计算？
- 哪些工具可以让 LLM 自主调用，哪些必须先让人确认？
- Tool result 应该返回原始数据、摘要，还是结构化 JSON？
- 如何评估 LLM 是不是“该用工具时用工具，不该用时不用”？

## Related

- [Agentic design patterns](../../concepts/agentic-design-patterns.md)
- [Tool use](../../concepts/tool-use.md)
- [External feedback](../../concepts/external-feedback.md)
- [Task decomposition](../../concepts/task-decomposition.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 新增 Module 3 README。
- [x] 新增 tool use concept card。
- [x] 下一节 `Creating a tool` 后，补充函数如何变成 tool。
- [ ] `Tool syntax` 后，补充 schema / 参数设计细节。
