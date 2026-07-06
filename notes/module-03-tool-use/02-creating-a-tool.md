---
course: Agentic AI
module: 3
lesson: 2
title: Creating a Tool
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/79cpry/creating-a-tool
watched_at: 2026-07-06
reviewed_at:
tags: [tool-use, function-calling, tool-call-loop, arguments, tool-result]
---

# Creating a Tool

## TL;DR

LLM 并不会真的直接执行函数。它会输出一个特定格式的 tool call request，开发者写的系统代码负责识别这个请求、调用真实函数、拿到结果，再把结果放回对话上下文，让 LLM 继续生成回答。现代 LLM 有更标准的 tool-call syntax，但底层流程仍然是这个循环。

## Core Ideas

- Tool 本质上是开发者实现的代码或函数，例如 `getCurrentTime`。
- 要让 LLM 使用工具，需要告诉它：有哪些工具、什么时候用、如何表达调用请求。
- 早期做法是让 LLM 输出类似 `FUNCTION: getCurrentTime` 的文本，再由开发者代码解析这段文本。
- 真正调用函数的是外部系统/开发者代码，不是 LLM 自己。
- 工具结果必须被写回 conversation history，LLM 才能基于结果继续回答。
- 工具可以有 arguments，例如 `getCurrentTime(Pacific/Auckland)`。
- LLM 可能调用一个工具后继续调用另一个工具，所以 tool use 是一个循环，而不是固定的一次调用。

## Tool Call Loop

```text
1. Developer implements a function/tool.
2. Developer tells the LLM the tool exists and how to request it.
3. User asks a question.
4. LLM outputs a tool call request.
5. Developer/system code parses the request.
6. System calls the real function with arguments.
7. System sends the tool result back to the LLM.
8. LLM uses the result to answer or request another tool call.
```

可以把它理解成：

```text
LLM: "我需要调用 getCurrentTime"
System: "好的，我替你调用。结果是 8 a.m."
LLM: "现在是 8 a.m."
```

## Why This Feels Weird At First

LLM 本质上是在生成 tokens。它不是 Python runtime，也不是数据库引擎。所以“LLM 调用函数”这句话容易误导。

更准确的说法是：

```text
LLM requests a function call.
The surrounding application executes the function.
The application returns the result to the LLM.
```

这对理解 agent 架构很重要：真正有权限、有副作用、能访问外部系统的是应用层代码，不是模型本身。

## Example 1: No Arguments

如果工具是：

```python
def get_current_time():
    ...
```

用户问“现在几点？”，LLM 可以输出一个特殊格式，表示它想调用这个函数。

早期 prompt engineering 可能让它输出：

```text
FUNCTION: getCurrentTime
```

然后 developer/system code 做三件事：

1. 识别 `FUNCTION:` 这个标记。
2. 找到 `getCurrentTime` 对应的真实函数。
3. 调用函数并把结果放回对话。

## Example 2: With Arguments

更真实的工具通常需要参数，比如：

```python
def get_current_time(timezone):
    ...
```

用户问“New Zealand 现在几点？”时，LLM 不只要选择 `getCurrentTime`，还要给出正确参数，比如 `Pacific/Auckland`。

早期文本格式可能长这样：

```text
FUNCTION: getCurrentTime Pacific/Auckland
```

这说明 tool use 不只是“选工具”，还包括“填参数”。参数填错，工具本身再正确也会返回错误或不相关结果。

## Modern Tool Syntax

课程提醒：今天主流 LLM 通常已经被训练成原生支持 tool calls，不需要开发者再用 `FUNCTION:` 这种笨重文本格式手写解析。

不过理解旧方式仍然有价值，因为它暴露了底层机制：

- LLM 生成 tool call request。
- 应用层执行工具。
- 工具结果回到 LLM。
- LLM 继续生成。

下一节会讲现代 tool syntax，也就是更结构化、更可靠的表达方式。

## Design Takeaways

- 不要把 tool use 理解成“模型有权限直接操作外部系统”；权限和执行都在应用层。
- Tool call request 要有稳定格式，否则系统难以解析。
- 有参数的工具更容易出错，所以参数名、类型、枚举值和说明很重要。
- Tool result 必须进入 conversation history，否则 LLM 不知道工具执行结果。
- 需要记录每次 tool call、arguments、tool result 和 error，方便 debug。
- 现代 function calling / tool syntax 本质上是在规范化这套 request -> execute -> return loop。

## Failure Modes

- LLM 输出的 tool call 格式不稳定，parser 识别失败。
- LLM 选对工具但传错 argument，例如时区、日期、表名、字段名。
- System 没有把 tool result 正确放回上下文，导致 LLM 继续猜。
- 工具执行失败，但错误信息没有清楚反馈给 LLM。
- 多轮工具调用时没有记录 history，后续步骤丢失上下文。
- 误以为 LLM 自己能执行函数，忽略应用层权限和安全控制。

## Implementation Hooks

- Tool registry: 建一个 tool name -> real function 的映射表。
- Parser / tool syntax: 识别 LLM 想调用哪个工具和参数。
- Argument validation: 检查参数类型、必填字段、枚举值和安全边界。
- Execution layer: 真正执行函数/API/database query。
- Tool result message: 把结果以清晰格式放回 LLM context。
- Logging: 保存 tool name、arguments、result、error、latency。

## My Questions

- 数据分析 agent 的 tool registry 应该怎么组织：按数据源、按任务类型，还是按权限等级？
- SQL 工具的 arguments 应该让 LLM 直接给 SQL，还是给更结构化的 query plan？
- Tool result 如果很长，比如返回很多 rows，应该怎么截断或摘要？
- 当参数有歧义时，LLM 应该追问用户，还是选择默认值？

## Related

- [What are tools?](01-what-are-tools.md)
- [Tool use](../../concepts/tool-use.md)
- [External feedback](../../concepts/external-feedback.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 3 README。
- [x] 补充 tool-use concept card 的 tool call loop 和 arguments。
- [ ] 下一节 `Tool syntax` 后，补充现代结构化 tool call schema。
- [ ] 后续 lab 时记录 function -> tool 的最小实现。
