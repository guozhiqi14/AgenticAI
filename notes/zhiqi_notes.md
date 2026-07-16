# Zhiqi Notes

这份笔记记录学习过程中出现的横向/通用问题。它不按课程小节整理，而是保存那些以后很可能反复遇到的概念困惑、代码机制和工程洞察。

---

## Tool Calling: 为什么要 append assistant tool call 和 tool result？

Date: 2026-07-07
Context: Module 3 Ungraded Lab: Turning functions into tools
Source notebook: `references/upstream/agentic_ai_andrew/Module3/ungraded_labs/M3_UGL_1/M3_UGL_1.ipynb`

### 我当时的困惑

在手动处理 tool call 的代码里，有两次 `messages.append(...)`：

```python
messages.append(response.choices[0].message)
messages.append({
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": str(tool_result)
})
```

问题是：

- 每一次 `message append` 到底要 append 什么？
- 是不是要把历史上所有对话都 append 一遍？
- `response.choices[0]` 这个结构到底是什么？
- 这是 AISuite 定制结构，还是 LLM API 里比较通用的结构？
- 这个 block 和上面自动 tool call 的方式，本质区别是什么？

### 先给结论

`messages` 是应用层手动维护的“对话历史列表”。每次 `append` 不是把所有历史重新追加一遍，而是把刚刚新发生的对话事件追加进去。下一次调用 LLM 时，再把累计后的完整 `messages` 发给模型。

在 tool calling 里，最重要的新事件通常有两类：

1. Assistant message: 模型刚才生成的 tool call request。
2. Tool message: 应用层真正执行函数后返回的 tool result。

也就是说，这两次 append 对应的是：

```text
assistant: 我想调用 get_current_time 这个工具
tool: 这个工具执行完了，结果是 14:32:10
```

模型下一轮只有看到这两条消息，才能理解完整链路，然后生成最终回答。

### 最小代码背景

notebook 里先让 LLM 看到一个手写 tool schema：

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_current_time",
        "description": "Returns the current time as a string.",
        "parameters": {}
    }
}]
```

这表示：当前可用工具列表里有一个 function tool，名字叫 `get_current_time`，不需要参数。

然后第一次调用模型：

```python
response = client.chat.completions.create(
    model="openai:gpt-4o",
    messages=messages,
    tools=tools,
)
```

这一次调用的目的不是拿最终答案，而是让模型判断：

```text
用户问 "What time is it?"
我是否需要调用工具？
如果需要，调用哪个工具？传什么参数？
```

如果模型决定要调用工具，它不会自己执行 Python。它会在返回结果里生成一个结构化的 `tool_calls` 请求。

### `response.choices[0]` 的 high-level 结构

可以先把 `response` 想成这样：

```text
response
  choices
    [0]
      message
        role
        content
        tool_calls
```

`choices` 是一个 list，因为有些 API 支持一次生成多个候选答案。课程里通常只要一个候选，所以取第一个：

```python
choice = response.choices[0]
```

这个 `choice.message` 是模型这一次生成的 assistant message。它可能有两种常见形态。

第一种：模型直接回答用户：

```python
{
    "role": "assistant",
    "content": "The current time is 14:32:10."
}
```

第二种：模型不直接回答，而是请求调用工具：

```python
{
    "role": "assistant",
    "content": None,
    "tool_calls": [
        {
            "id": "call_xxx",
            "type": "function",
            "function": {
                "name": "get_current_time",
                "arguments": "{}"
            }
        }
    ]
}
```

这就是 `response.choices[0].message.tool_calls` 的来源。

### 哪些结构是通用的，哪些是 AISuite 包装的？

比较通用的 Chat Completions 风格结构：

```python
response.choices[0]
response.choices[0].message
response.choices[0].message.content
response.choices[0].message.tool_calls
```

这些结构不是某个业务场景定制出来的，而是很多 chat-completion / function-calling API 都会采用的返回形态。

AISuite 额外方便教学的部分是：

```python
choice.intermediate_messages
```

notebook 里的 `display_functions.pretty_print_chat_completion(response)` 会读取这个字段，把中间的 tool call 和 tool response 展示成 HTML。这个字段更像 AISuite 为了展示 tool-call loop 额外包装出来的 trace，不要默认所有 API 都有。

### 每次 append 的到底是什么？

一开始，`messages` 只有用户问题：

```python
messages = [
    {
        "role": "user",
        "content": "What time is it?"
    }
]
```

第一次调用 LLM 后，模型返回：

```python
response.choices[0].message
```

这条 message 的含义是：

```text
assistant: 我想调用 get_current_time 工具
```

所以第一条 append 是：

```python
messages.append(response.choices[0].message)
```

现在 `messages` 变成：

```python
[
    {"role": "user", "content": "What time is it?"},
    assistant_message_with_tool_call
]
```

然后应用层真正执行 Python 函数：

```python
tool_result = get_current_time()
```

这行才是真正执行工具。LLM 没有执行 Python，它只是请求调用。

工具执行完后，要把结果也追加进历史：

```python
messages.append({
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": str(tool_result)
})
```

现在 `messages` 变成：

```python
[
    {"role": "user", "content": "What time is it?"},
    assistant_message_with_tool_call,
    {
        "role": "tool",
        "tool_call_id": "call_xxx",
        "content": "14:32:10"
    }
]
```

这时再第二次调用 LLM：

```python
response2 = client.chat.completions.create(
    model="openai:gpt-4o",
    messages=messages,
    tools=tools,
)
```

模型看到完整上下文：

```text
user: What time is it?
assistant: 我想调用 get_current_time
tool: 14:32:10
```

于是它才能生成最终自然语言回答：

```text
The current time is 14:32:10.
```

### 为什么不是只 append tool result？

如果只追加工具结果：

```python
messages.append({
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": str(tool_result)
})
```

模型下一轮会看到一个缺上下文的 tool message：

```text
tool: 14:32:10
```

但它不知道这个工具结果对应哪一次 assistant tool call。尤其当模型一次请求多个工具时，这会更乱。

所以必须保留这条 assistant message：

```python
messages.append(response.choices[0].message)
```

它的作用是记录：

```text
刚才模型确实请求过这个工具调用。
```

然后 tool message 里的 `tool_call_id` 用来对齐：

```text
这个工具结果，是对 call_xxx 这次工具请求的响应。
```

### `tool_call_id` 为什么重要？

简单任务里只有一个 tool call，看起来 `tool_call_id` 好像多余。但真实场景里，模型可能一次请求多个工具：

```text
call_1 -> get_weather_from_ip
call_2 -> get_current_time
```

工具返回时，系统需要告诉模型：

```text
这个结果属于 call_1
那个结果属于 call_2
```

否则模型只看到一堆结果，很难稳定知道哪个结果对应哪个请求。

所以 `tool_call_id` 是 tool-call protocol 里的“对账编号”。

### 自动模式和手动模式的本质区别

上面 notebook 里先用了自动模式：

```python
response = client.chat.completions.create(
    model="openai:gpt-4o",
    messages=messages,
    tools=[get_current_time],
    max_turns=5
)
```

这里 AISuite 自动帮你做了：

```text
1. 从 Python function / docstring 生成 tool schema
2. 把 tool schema 发给 LLM
3. 读取模型返回的 tool_calls
4. 执行本地 Python 函数
5. 把 tool result 追加回 messages
6. 再次调用 LLM
7. 重复直到最终回答或达到 max_turns
```

而手动模式是：

```python
response = client.chat.completions.create(...)

if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    args = json.loads(tool_call.function.arguments)

    tool_result = get_current_time()

    messages.append(response.choices[0].message)
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": str(tool_result)
    })

    response2 = client.chat.completions.create(...)
```

这段代码把自动模式背后的过程摊开了。

本质区别：

```text
自动模式：library 帮你管理 tool-call loop
手动模式：你自己管理 tool-call loop
```

### Under the hood: tool calling 不是魔法

tool calling 的底层机制可以理解成一个协议：

```text
LLM 负责生成结构化请求：
  tool name = get_current_time
  arguments = {}

应用层负责真实执行：
  get_current_time()

应用层再把结果写回 messages：
  role = tool
  tool_call_id = call_xxx
  content = 14:32:10

LLM 再根据完整 messages 生成最终回答。
```

关键洞察：

- LLM 不拥有真实执行权限。
- 函数/API/数据库/文件系统权限都在应用层。
- LLM 只是提出请求。
- 应用层可以选择执行、拒绝、改写、校验、记录或要求人工确认。

这就是 tool calling 在业务系统里可控的原因。

### 业务系统里的迁移理解

假设以后做一个数据分析 agent，工具可能是：

```python
def lookup_table_schema(table_name: str):
    ...

def run_sql_query(sql: str):
    ...

def get_metric_definition(metric_name: str):
    ...

def create_chart(data_path: str, chart_type: str):
    ...
```

用户问：

```text
帮我看一下上周新用户留存为什么下降。
```

模型可能先请求：

```text
lookup_table_schema("user_retention")
```

应用层执行后返回 schema。模型再请求：

```text
get_metric_definition("new_user_retention")
```

再生成 SQL：

```text
run_sql_query(...)
```

每一步都要写进 `messages`，因为下一步推理依赖上一步工具结果。

这说明 `messages` 不是普通聊天记录那么简单，它是 agent runtime 的状态账本：

```text
用户意图
模型决策
工具调用请求
工具执行结果
下一步模型决策
最终回答
```

### 真实生产代码还需要补什么？

notebook 的手动 block 是教学最小版，不是完整生产版。真实系统一般还需要：

Tool registry:

```python
tool_registry = {
    "get_current_time": get_current_time,
    "write_txt_file": write_txt_file,
    "get_weather_from_ip": get_weather_from_ip,
}
```

动态选择函数：

```python
tool_name = tool_call.function.name
tool_func = tool_registry[tool_name]
args = json.loads(tool_call.function.arguments)
tool_result = tool_func(**args)
```

还需要：

- 参数校验：模型传来的参数类型、路径、SQL、日期范围是否安全。
- 错误处理：工具失败时如何把错误返回给模型。
- 权限控制：写文件、发邮件、改数据库、下单等副作用工具不能无条件执行。
- 日志记录：记录 tool name、arguments、result、latency、error。
- 循环上限：避免模型一直调用工具，类似 `max_turns`。
- 人工确认：高风险动作先让人确认。

### 我应该记住的判断标准

当我看到 tool calling 代码时，先问：

1. `messages` 里是否保留了完整链路？
2. assistant 的 tool call message 是否被 append？
3. tool result 是否用 `role: "tool"` 写回？
4. `tool_call_id` 是否能对齐请求和结果？
5. 下一次调用 LLM 时，是否传入累计后的完整 `messages`？
6. 工具是否真的在应用层执行，而不是误以为模型在执行？
7. 如果工具有副作用，是否有校验、权限和日志？

### 一句话总结

`messages.append(...)` 的本质不是“随便保存聊天记录”，而是在维护 agent 的状态账本。模型通过 `response.choices[0].message.tool_calls` 提出工具调用请求，应用层执行工具，再把 assistant tool call 和 tool result 都写回 `messages`。下一次调用 LLM 时，模型才能基于完整历史继续推理并生成最终回答。

---

## Messages / Roles / API Return: 不要把本地历史和 API 返回混在一起

Date: 2026-07-09
Context: Module 3 Graded Lab: Research agent
Source notebook: `references/upstream/agentic_ai_andrew/Module3/graded_labs/research_agent/GL-M3.ipynb`
Related docs:

- OpenAI Chat Completions API reference: `https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create`
- OpenAI Function calling guide: `https://developers.openai.com/api/docs/guides/function-calling`

### 我当时的困惑

在这个 loop 里：

```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
    tools=functions,
    tool_choice="auto",
)

msg = response.choices[0].message
messages.append(msg)
```

我一开始容易把两个东西混在一起：

- `messages`: 是不是 API 返回回来的一整个历史？
- `response.choices[0].message`: 是不是包含了 user 原始问题和 assistant 回复？
- `choices`: 是不是像 `messages` 一样，是一组 user / assistant 对话？

### 最重要的结论

`messages` 是应用层自己维护的完整上下文；`response.choices[0].message` 只是模型本轮新生成的一条 assistant message。

也就是说：

```text
messages = 你发给模型的 conversation so far
response.choices[0].message = 模型这次新生成的一条 assistant message
```

API 不会在返回值里自动把原始 user 问题、历史 assistant 回复、tool result 全部打包回来。历史是否完整，取决于应用代码有没有把每一步正确 append 到 `messages`。

### 第一次调用时到底发生什么？

第一次调用前，通常是应用自己构造：

```python
messages = [
    {
        "role": "system",
        "content": "You are a research assistant."
    },
    {
        "role": "user",
        "content": "Radio observations of recurrent novae"
    }
]
```

然后调用：

```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
    tools=functions,
    tool_choice="auto",
)
```

返回的不是：

```python
[
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
]
```

而更像：

```python
response = {
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "...",
                "tool_calls": None
            }
        }
    ]
}
```

所以 `response.choices[0].message` 不是完整聊天记录，只是新增的 assistant message。要让下一轮模型看到这条 assistant message，必须自己做：

```python
messages.append(response.choices[0].message)
```

append 后，本地的 `messages` 才变成：

```python
[
    {"role": "system", "content": "..."},
    {"role": "user", "content": "Radio observations of recurrent novae"},
    {"role": "assistant", "content": "..."}
]
```

### `choices` 和 `messages` 不是一回事

这两个名字很容易误导：

```text
messages = 对话历史列表
choices = 本次模型生成的候选答案列表
```

如果 API 返回：

```python
response.choices[0].message
response.choices[1].message
```

它们通常表示两个候选 assistant response，而不是：

```text
choices[0] = user message
choices[1] = assistant message
```

课程里通常只需要一个候选，所以取：

```python
msg = response.choices[0].message
```

### 各种 role 的本质含义

可以把 `role` 理解成“这条消息是谁说的，以及它在协议里承担什么职责”。

`system`

```text
系统 / 开发者给模型的高优先级行为约束。
```

例如：

```python
{"role": "system", "content": "You are a research assistant. Cite sources."}
```

它不是用户具体问题，而是给整个 workflow 定调：身份、语气、边界、格式、是否使用工具、是否引用来源。

`user`

```text
用户或应用代表用户发给模型的新任务 / 新要求。
```

例如：

```python
{"role": "user", "content": "Radio observations of recurrent novae"}
```

如果用户没有追问，通常只有一条 `role="user"`。后续 agent 自己查资料、调用工具、生成报告，这些都不是新的 user message。

但不是语法上只能有一个 user。多轮聊天里，用户每追问一次，应用就会追加一条新的 user message：

```python
[
    {"role": "user", "content": "先查 recurrent novae"},
    {"role": "assistant", "content": "这里是初步结果..."},
    {"role": "user", "content": "再重点看 radio observations"}
]
```

`assistant`

```text
模型本轮生成的输出。
```

普通聊天里，它可能是自然语言：

```python
{"role": "assistant", "content": "Here is the report..."}
```

tool calling 里，它也可能不是最终回答，而是工具调用请求：

```python
{
    "role": "assistant",
    "content": None,
    "tool_calls": [
        {
            "id": "call_xxx",
            "type": "function",
            "function": {
                "name": "arxiv_search_tool",
                "arguments": "{\"query\": \"Radio observations of recurrent novae\"}"
            }
        }
    ]
}
```

所以要记住：assistant 不等于“最终答案”。assistant 的本质是“模型这一步产生了什么”。它可能是回答，也可能是 action request。

`tool`

```text
应用层真正执行工具后，返回给模型的外部结果。
```

例如：

```python
{
    "role": "tool",
    "tool_call_id": "call_xxx",
    "name": "arxiv_search_tool",
    "content": "[{\"title\": \"...\", \"summary\": \"...\"}]"
}
```

这条消息不是模型写的，而是 Python 代码、后端服务、数据库、搜索 API 等外部系统的结果。模型下一轮会读这条 tool message，再决定继续调用工具还是生成最终回答。

### 单个用户问题下的消息流

如果用户只问一次，后面没有追问，那么 `messages` 大概会像这样增长：

```text
system: 你是研究助手，要引用来源
user: Radio observations of recurrent novae
assistant: 我需要调用 arxiv_search_tool(...)
tool: arxiv 返回论文列表
assistant: 根据论文列表，生成研究报告
```

如果模型继续需要更多资料，可能是：

```text
system
user
assistant: call arxiv_search_tool(...)
tool: arxiv results
assistant: call tavily_search_tool(...)
tool: web results
assistant: final report
```

这里没有新的 `user`，因为用户没有提出新要求。中间的推进都是 agent 在完成同一个 user task。

### 业务场景里的理解

假设做一个数据分析 agent，用户问：

```text
帮我看一下上周新用户留存为什么下降。
```

第一版 `messages`：

```python
[
    {"role": "system", "content": "You are a data analyst agent."},
    {"role": "user", "content": "帮我看一下上周新用户留存为什么下降。"}
]
```

模型可能返回一条 assistant tool call：

```python
{
    "role": "assistant",
    "content": None,
    "tool_calls": [
        {
            "id": "call_1",
            "function": {
                "name": "lookup_table_schema",
                "arguments": "{\"table_name\": \"user_retention\"}"
            }
        }
    ]
}
```

应用层 append 这条 assistant message，然后执行工具，把结果写回：

```python
{
    "role": "tool",
    "tool_call_id": "call_1",
    "content": "{\"columns\": [\"dt\", \"channel\", \"new_users\", \"d1_retention_rate\"]}"
}
```

下一轮模型看到完整历史，可能继续请求：

```text
assistant: call get_metric_definition("new_user_retention")
tool: metric definition
assistant: call run_sql_query(...)
tool: query result
assistant: final diagnosis
```

这时 `messages` 像一个分析过程账本：

```text
用户要解决什么业务问题
模型判断先查什么
工具返回了什么事实
模型根据事实又做了什么下一步
最终给出什么结论
```

对数据分析工作来说，这个账本非常重要。它不仅能让模型继续推理，也能让人 debug：到底是 metric 口径错了、SQL 错了、工具结果错了，还是最后解释错了。

### 常见误区

误区 1：以为 `response.choices[0].message` 包含完整历史。

实际：它只是一条新 assistant message。完整历史在本地 `messages` 里。

误区 2：以为 `choices` 是 user / assistant 对话列表。

实际：`choices` 是本次生成的候选结果列表。通常 `choices[0]` 就够了。

误区 3：以为没有自然语言 content 的 assistant message 没有用。

实际：带 `tool_calls` 的 assistant message 非常关键，它记录了模型请求了哪个工具、参数是什么、`tool_call_id` 是什么。

误区 4：只 append tool result，不 append assistant tool call。

实际：下一轮模型会看到一个缺少前因后果的 tool message，不知道这个结果对应哪次工具请求。

误区 5：把 tool call 当成工具已经执行。

实际：tool call 只是模型提出请求；真正执行工具的是应用层代码。

### Practice: 看 tool-calling 代码时先检查什么

看到任何类似下面的代码：

```python
response = client.chat.completions.create(...)
msg = response.choices[0].message
messages.append(msg)
```

先问：

1. `messages` 是在哪里初始化的？里面有没有 system / user？
2. `response.choices[0].message` 有没有被 append 回本地历史？
3. 如果有 `msg.tool_calls`，应用层有没有执行对应工具？
4. tool result 有没有用 `role="tool"` 写回？
5. tool result 有没有带正确的 `tool_call_id`？
6. 下一次调用模型时，传入的是不是累计后的完整 `messages`？
7. 如果用户没有追问，为什么后续没有新的 `role="user"`？
8. 如果工具有副作用，是否有参数校验、权限控制、日志和人工确认？

### 一句话总结

`messages` 是应用维护的 agent 状态账本；`response.choices[0].message` 是模型本轮新写的一条 assistant 事件。单个用户问题下，通常只有一条 user message，后面是 assistant / tool / assistant 的循环。tool calling 的关键不是“模型会执行函数”，而是“模型提出结构化请求，应用执行工具，再把结果写回 messages，让模型继续推理”。

---

## HTTP / URL / REST route: tool 背后到底怎么和后端说话？

Date: 2026-07-07
Context: Module 3 Ungraded Lab: Email assistant workflow
Source notebook: `references/upstream/agentic_ai_andrew/Module3/ungraded_labs/M3_UGL_2/M3_UGL_2.ipynb`
Source tools: `references/upstream/agentic_ai_andrew/Module3/ungraded_labs/M3_UGL_2/email_tools.py`

### 我当时的困惑

在 notebook 里看到这句话：

```text
Each tool wraps a REST route.
```

我一开始听成了 `test route`，后来确认应该是 `REST route`。但新的问题是：

- REST route 是什么？
- URL + HTTP method 又是什么意思？
- HTTP 不是浏览器里输入的网址吗，为什么 Python、App、手机端也会用？
- 如果我是数据分析师，只想快速理解开发底层逻辑和架构，需要理解到哪个 level？

### 先给结论

HTTP 不是“网址”本身，而是一套客户端和服务器之间通信的规则。

浏览器、手机 App、桌面 App、Python 程序、后端服务、LLM tool 背后的函数，都可以通过 HTTP 和服务器说话。

最核心的模型是：

```text
client sends request
server returns response
```

也就是：

```text
客户端发请求 -> 服务器处理 -> 服务器返回结果
```

在这个 email agent lab 里，完整链路是：

```text
用户自然语言请求
  -> LLM 选择 email tool
  -> Python tool 发 HTTP request
  -> FastAPI backend 收到 request
  -> backend 操作 SQLite 数据库
  -> backend 返回 HTTP response
  -> Python tool 把结果交回 LLM
  -> LLM 决定下一步或总结结果
```

### HTTP 到底是什么？

HTTP 可以理解成互联网应用跟后端服务器说话的通用语言规则。

它不是某个具体网站，也不是某个浏览器专属能力。浏览器只是最常见的 HTTP client 之一。

能发 HTTP request 的东西包括：

- Chrome / Safari 浏览器。
- 手机 App，比如外卖、地图、天气、微信、淘宝。
- 桌面 App。
- Python 脚本，比如 `requests.get(...)`。
- 后端服务之间的调用。
- LLM tool 背后的业务函数。

比如你打开天气 App，界面上没有输入网址，但 App 背后可能会发：

```text
GET /weather/current?city=shenzhen
```

服务器返回天气数据，App 再把 JSON 数据画成你看到的天气卡片。

### URL 是什么？

URL 是地址，告诉客户端要找哪个服务器、哪个资源。

比如：

```text
https://api.example.com/emails/3
```

可以拆成：

```text
https://          使用 HTTPS 协议
api.example.com   服务器地址
/emails/3         具体资源路径
```

在本地开发里，服务器可能跑在自己电脑上：

```text
http://127.0.0.1:8000
```

如果再加上路径：

```text
http://127.0.0.1:8000/emails
http://127.0.0.1:8000/emails/unread
http://127.0.0.1:8000/emails/search?q=lunch
http://127.0.0.1:8000/emails/3
```

可以理解成不同业务柜台：

```text
/emails                邮件列表柜台
/emails/unread         未读邮件柜台
/emails/search?q=...   搜索邮件柜台
/emails/3              第 3 封邮件这个资源
/send                  发邮件柜台
```

### HTTP method 是什么？

URL 告诉服务器“我要操作哪个东西”，HTTP method 告诉服务器“我要做什么动作”。

先记住这几个就够了：

```text
GET     读取 / 查看
POST    创建 / 提交
PATCH   修改一部分
DELETE  删除
```

放到 email lab 里：

```text
GET /emails
查看所有邮件

GET /emails/unread
查看未读邮件

POST /send
发送一封新邮件

PATCH /emails/3/read
把第 3 封邮件标记为已读

DELETE /emails/3
删除第 3 封邮件
```

所以：

```text
URL = 对象
HTTP method = 动作
```

这也是为什么我应该把 `URL + HTTP method` 放在一起理解。只有 URL 不够，因为同一个资源可以有不同动作：

```text
GET /emails/3
查看第 3 封邮件

DELETE /emails/3
删除第 3 封邮件
```

### HTTP request 具体包含什么？

一个 HTTP request 通常由这几部分组成：

```text
1. Method: 我要做什么动作
2. URL / path: 我要操作哪个资源
3. Headers: 附加说明，比如身份、格式
4. Body: 真正提交的数据，比如邮件正文
```

查看所有邮件时，request 可以很简单：

```text
GET /emails
```

发送邮件时，需要带 body：

```text
POST /send
Content-Type: application/json

{
  "recipient": "boss@email.com",
  "subject": "Follow up",
  "body": "Thanks for your email."
}
```

这里：

```text
POST /send
```

表示我要创建/发送一封邮件。

JSON body 表示邮件内容。

### HTTP response 具体包含什么？

服务器收到 request 后，会返回 response。一个 response 通常包括：

```text
1. Status code: 成功还是失败
2. Headers: 返回内容的说明
3. Body: 真正返回的数据
```

比如成功返回：

```text
200 OK

[
  {
    "id": 1,
    "sender": "boss@email.com",
    "recipient": "you@email.com",
    "subject": "Quarterly Report",
    "read": false
  }
]
```

常见状态码：

```text
200 成功
201 创建成功
400 请求格式有问题
401 没登录 / 没身份凭证
403 登录了但没权限
404 找不到资源
500 服务器内部错误
```

### `Each tool wraps a REST route` 是什么意思？

在这个 notebook 里，`email_tools.py` 里的每个 Python function 都是一个 tool。它们表面上是普通 Python 函数，但里面真正做的事是调用某个后端 route。

比如：

```python
def list_all_emails() -> list:
    return requests.get(f"{BASE_URL}/emails").json()
```

这表示：

```text
Python tool: list_all_emails()
HTTP request: GET /emails
后端动作: 返回所有邮件
```

再比如：

```python
def search_emails(query: str) -> list:
    return requests.get(
        f"{BASE_URL}/emails/search",
        params={"q": query}
    ).json()
```

如果调用：

```python
search_emails("lunch")
```

背后相当于发：

```text
GET /emails/search?q=lunch
```

再比如删除邮件：

```python
def delete_email(email_id: int) -> dict:
    return requests.delete(f"{BASE_URL}/emails/{email_id}").json()
```

如果：

```python
delete_email(3)
```

背后相当于发：

```text
DELETE /emails/3
```

所以 `Each tool wraps a REST route` 的意思是：

```text
每个 tool 是一个 Python 外壳。
这个外壳把一个后端 API route 包起来。
LLM 看起来是在调用 Python 函数。
Python 函数实际是在通过 HTTP 请求操作后端系统。
```

### REST route 是什么？

这里可以先不深究 REST 的完整理论，只要理解成：

```text
REST route = 后端暴露出来的一个资源操作入口
```

它通常长这样：

```text
GET /emails
POST /send
GET /emails/search?q=...
PATCH /emails/{email_id}/read
DELETE /emails/{email_id}
```

其中：

```text
GET / POST / PATCH / DELETE = 动作
/emails / /send / /emails/{id} = 资源路径
```

在 `email_server/email_service.py` 里，后端就是这样定义 route 的：

```python
@app.get("/emails")
def list_emails(...):
    ...
```

```python
@app.post("/send")
def send_email(...):
    ...
```

```python
@app.delete("/emails/{email_id}")
def delete_email(...):
    ...
```

这些 route 就像后端开放出来的一组业务按钮。前端、Python tool 或其他服务都可以通过 HTTP request 按这些按钮。

### 和数据库的关系

作为数据分析师，可以这样类比：

```text
SQL: 直接问数据库
API: 先问后端服务，后端服务再查数据库
```

比如：

```python
requests.get(f"{BASE_URL}/emails/search", params={"q": "lunch"})
```

背后可能对应后端里的数据库查询：

```python
db.query(Email).filter(
    (Email.subject.ilike("%lunch%")) |
    (Email.body.ilike("%lunch%")) |
    (Email.sender.ilike("%lunch%"))
)
```

真实业务系统通常不会让前端、手机 App 或 LLM 直接查数据库，而是让它们调用 API。

因为 API 可以做这些事：

- 验证用户身份。
- 控制权限。
- 校验参数格式。
- 限制能查哪些数据。
- 记录日志。
- 防止危险操作。
- 把底层数据库结构隐藏起来。

### 这个 notebook 的完整操作链路

用户说：

```text
Delete the happy hour email
```

更合理的 agent 执行链路是：

```text
LLM:
我需要先找到 Happy Hour 邮件

Python tool:
search_emails("Happy Hour")

HTTP request:
GET /emails/search?q=Happy Hour

FastAPI backend:
查 SQLite 数据库

HTTP response:
返回匹配邮件，包括真实 email_id

LLM:
我现在知道要删除哪个 id

Python tool:
delete_email(email_id)

HTTP request:
DELETE /emails/{email_id}

FastAPI backend:
删除数据库里的这封邮件

LLM:
总结给用户：已经删除
```

注意：不要直接依赖 notebook markdown 里的固定 `id`。这个 lab 的后端启动时会 shuffle 初始邮件，所以更稳的做法是先 search，再拿当前返回的真实 id 去 delete。

### 我应该理解到哪个 level？

作为数据分析师，如果目标是快速理解开发底层逻辑和 agent 架构，先掌握到这 5 层就够用了：

```text
Level 1: HTTP 是 request -> response
Level 2: request = method + URL/path + headers + body
Level 3: response = status code + headers + body/JSON
Level 4: 能看懂 Python requests 调的是哪个 API route
Level 5: 能把业务动作翻译成 API 调用链
```

暂时不需要先深挖：

```text
TCP/IP
socket
TLS 握手
HTTP/2 frame
负载均衡细节
浏览器渲染机制
```

这些以后如果做后端或系统架构，再慢慢补。

### 和 agentic AI 的关系

这个知识点和 tool use 的关系非常直接。

在 agent 系统里：

```text
LLM 不直接操作数据库。
LLM 也不直接拥有后端权限。
LLM 只是请求调用工具。
Python tool 通过 HTTP request 调后端 API。
后端 API 才真正执行业务逻辑。
```

因此，一个业务 agent 的真实能力来自：

```text
暴露给 LLM 的工具
  -> 工具包装的 API route
  -> 后端允许执行的业务逻辑
  -> 数据库或外部系统里的真实状态变化
```

这也是为什么工具设计、API 权限、参数校验和日志都很重要。模型再聪明，如果没有暴露 `delete_email` 工具，它就不能删除邮件；如果暴露了删除工具但没有权限控制，就可能带来真实业务风险。

### 一句话总结

HTTP 是客户端和服务器之间的通用通信规则。URL 告诉服务器要操作哪个资源，HTTP method 告诉服务器要做什么动作，request 带着必要信息发出去，response 带着结果返回。`Each tool wraps a REST route` 的意思是：LLM 调用的 Python tool 背后，其实是在通过 HTTP 请求调用某个后端 API route，让 agent 能间接操作真实业务系统。

---

## Module 5 Research Agent: executor 如何调度 sub-agent？

Date: 2026-07-17
Context: Module 5 Graded Lab: Research Agent / Multi-agent Workflow
Source notebook: `references/upstream/agentic_ai_andrew/Module5/graded_labs/research-agent/GL-M5.ipynb`
Related file: `references/upstream/agentic_ai_andrew/Module5/graded_labs/research-agent/research_tools.py`

### 我当时的困惑

这个 notebook 里有 `planner_agent`、`research_agent`、`writer_agent`、`editor_agent` 和 `executor_agent`。

一开始容易误解的是：

- sub-agent 是不是像 tool 一样被模型直接调用？
- `executor_agent` 里面看起来只是 `history.append(...)`，那它到底在哪里真正执行了 sub-agent？
- `research_agent` 查到的内容是 paper 全文，还是只是摘要和链接？
- 如果只是摘要，writer_agent 是怎么拿到这些信息的？
- agent 之间到底有没有互相发 message？

### 先给结论

这个 lab 里的 sub-agent 不是 OpenAI function calling 意义上的 tool。它们本质上是普通 Python 函数：

```python
def research_agent(task: str, ...):
    ...

def writer_agent(task: str, ...):
    ...

def editor_agent(task: str, ...):
    ...
```

真正让它们能被调度的是一个 Python dict：

```python
agent_registry = {
    "research_agent": research_agent,
    "editor_agent": editor_agent,
    "writer_agent": writer_agent,
}
```

executor 的真实调用发生在这一行：

```python
output = agent_registry[agent_name](enriched_task)
```

如果 `agent_name == "research_agent"`，这行就等价于：

```python
output = research_agent(enriched_task)
```

所以：

```text
sub-agent 是被 executor 当 Python function 调用的。
tool 是被 research_agent 内部的 LLM/tool loop 调用的。
history 不是执行器本身，而是跨 agent 的上下文传递介质。
```

### 整体架构

这个 notebook 的 end-to-end 结构可以理解成：

```text
topic
  -> planner_agent 生成 plan steps
  -> executor_agent 逐步执行每个 step
      -> LLM router 判断该交给哪个 sub-agent
      -> Python registry 调用对应 sub-agent 函数
      -> sub-agent 内部再调用 LLM
      -> research_agent 可以进一步使用 tools
      -> output 被 append 到 history
  -> 最后一步 output 被当成 Markdown report 显示
```

更具体一点：

```text
executor_agent
  -> agent_registry["research_agent"](enriched_task)
      -> research_agent(...)
          -> client.chat.completions.create(..., tools=[...])
              -> arxiv_search_tool(...)
              -> tavily_search_tool(...)
              -> wikipedia_search_tool(...)
          -> return research summary
  -> history.append((step, "research_agent", output))
```

而 `writer_agent` / `editor_agent` 没有工具，链路更短：

```text
executor_agent
  -> writer_agent(enriched_task)
      -> LLM writes based on provided context
      -> return text
  -> history.append(...)
```

### 一个具体 end-to-end 例子

假设用户给的 topic 是：

```text
The ensemble Kalman filter for time series forecasting
```

`planner_agent(topic)` 可能返回一个 plan：

```python
[
    "Search for key academic papers on ensemble Kalman filter for time series forecasting.",
    "Summarize the main methods and applications from the gathered sources.",
    "Draft a structured technical research report.",
    "Critique the draft for clarity, structure, and missing citations.",
    "Generate a final Markdown document with the complete research report."
]
```

接下来调用：

```python
executor_history = executor_agent(steps)
```

### Step 1: executor 先做 routing，不直接做研究

第一步 instruction 是：

```text
Search for key academic papers on ensemble Kalman filter for time series forecasting.
```

`executor_agent` 不会自己搜索。它先问一次 LLM：

```text
Given the following instruction, identify which agent should perform it and extract the clean task.

Return only a valid JSON object with two keys:
- "agent": one of ["research_agent", "editor_agent", "writer_agent"]
- "task": a string with the instruction that the agent should follow
```

LLM 可能返回：

```json
{
  "agent": "research_agent",
  "task": "Search arXiv, Wikipedia, and the web for key sources on ensemble Kalman filter for time series forecasting."
}
```

然后 executor 解析 JSON：

```python
raw_content = response.choices[0].message.content
cleaned_json = clean_json_block(raw_content)
agent_info = json.loads(cleaned_json)

agent_name = agent_info["agent"]
task = agent_info["task"]
```

此时：

```python
agent_name == "research_agent"
```

### Step 2: executor 通过 registry 真实调用 sub-agent

executor 会构造 `enriched_task`：

```python
context = "\n".join([
    f"Step {j+1} executed by {a}:\n{r}"
    for j, (s, a, r) in enumerate(history)
])

enriched_task = f"""You are {agent_name}.

Here is the context of what has been done so far:
{context}

Your next task is:
{task}
"""
```

因为这是第一步，`history` 还为空，所以传给 `research_agent` 的大概是：

```text
You are research_agent.

Here is the context of what has been done so far:

Your next task is:
Search arXiv, Wikipedia, and the web for key sources on ensemble Kalman filter for time series forecasting.
```

真正执行发生在：

```python
output = agent_registry[agent_name](enriched_task)
```

因为 `agent_name` 是 `"research_agent"`，所以等价于：

```python
output = research_agent(enriched_task)
```

### Step 3: research_agent 内部再调用 tools

`research_agent` 里面会重新组织自己的 `messages`：

```python
messages = [{"role": "user", "content": prompt.strip()}]
```

并且把工具函数传给模型：

```python
tools = [
    research_tools.arxiv_search_tool,
    research_tools.tavily_search_tool,
    research_tools.wikipedia_search_tool
]
```

然后调用：

```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
    tools=tools,
    tool_choice="auto",
    max_turns=12
)
```

这里的意思是：

```text
research_agent 问 LLM: 这个 research task 怎么做？
LLM 决定要不要调用工具。
如果要调用工具，aisuite 帮忙执行对应 Python function。
工具结果返回给 LLM。
LLM 可以继续调用别的工具。
最多允许 12 轮 tool iteration。
最后 LLM 生成一个 research summary。
```

所以 `research_agent` 不是自己写死调用哪个工具，而是把工具能力交给 LLM 自动选择。

### research_agent 返回的是全文吗？

不是。

这个 notebook 里的 research tools 通常只返回 metadata、摘要、片段和 URL。

`arxiv_search_tool` 返回类似：

```python
{
    "title": "...",
    "authors": [...],
    "published": "...",
    "url": "...",
    "summary": "...",
    "link_pdf": "..."
}
```

它有 arXiv abstract 和 PDF link，但不会自动下载 PDF，也不会读取 paper 全文。

`tavily_search_tool` 返回类似：

```python
{
    "title": "...",
    "content": "...",
    "url": "..."
}
```

这里的 `content` 通常是搜索结果摘要或 snippet，不是完整网页正文。

`wikipedia_search_tool` 返回类似：

```python
{
    "title": "...",
    "summary": "...",
    "url": "..."
}
```

所以 `research_agent` 更准确地说是：

```text
搜索 + 摘要 agent
```

而不是：

```text
全文 paper reading agent
```

如果要真的读 paper，需要额外增加工具：

```text
download_pdf_tool
extract_pdf_text_tool
read_arxiv_pdf_tool
chunk_and_summarize_paper_tool
```

然后流程才能变成：

```text
search paper
  -> get PDF link
  -> download PDF
  -> extract text
  -> summarize methods / experiments / limitations
  -> pass structured evidence to writer
```

### Step 4: research_agent 的输出如何传给 writer_agent？

`research_agent` 最终会返回一个字符串，比如：

```text
Key sources found:
1. Evensen introduced the Ensemble Kalman Filter as a Monte Carlo approximation for nonlinear filtering...
   URL: ...
2. Later work applies EnKF to nonlinear state-space models and data assimilation...
   URL: ...
3. Time series forecasting applications use EnKF to update latent state estimates as observations arrive...
   URL: ...
```

executor 会把这个结果 append 到 `history`：

```python
history.append((step, agent_name, output))
```

此时 `history` 大概是：

```python
[
    (
        "Search for key academic papers on ensemble Kalman filter for time series forecasting.",
        "research_agent",
        "Key sources found:\n1. Evensen introduced..."
    )
]
```

下一步如果 planner step 是：

```text
Summarize the main methods and applications from the gathered sources.
```

executor 又会问 LLM router。LLM 可能返回：

```json
{
  "agent": "writer_agent",
  "task": "Summarize the main methods and applications from the gathered sources."
}
```

然后 executor 重新构造 context：

```python
context = "\n".join([
    f"Step {j+1} executed by {a}:\n{r}"
    for j, (s, a, r) in enumerate(history)
])
```

得到：

```text
Step 1 executed by research_agent:
Key sources found:
1. Evensen introduced the Ensemble Kalman Filter...
2. ...
```

最后传给 writer 的 `enriched_task` 是：

```text
You are writer_agent.

Here is the context of what has been done so far:
Step 1 executed by research_agent:
Key sources found:
1. Evensen introduced the Ensemble Kalman Filter...
2. ...

Your next task is:
Summarize the main methods and applications from the gathered sources.
```

所以 writer_agent 看到的是：

```text
前面 research_agent 的最终输出字符串
```

不是：

```text
工具原始 JSON
paper 全文
research_agent 的内部 tool-call trace
```

除非 research_agent 主动把这些内容写进自己的 final output。

### Step 5: writer_agent 如何基于 history 写 draft？

`writer_agent` 内部没有工具，它只是调用 LLM：

```python
messages = [
    {
        "role": "system",
        "content": "You are a writing agent specialized in generating well-structured academic or technical content."
    },
    {
        "role": "user",
        "content": enriched_task
    }
]
```

所以 writer 的信息来源就是 `enriched_task`。

如果 `enriched_task` 里只有很薄的摘要，writer 只能写出摘要级报告：

```text
高层背景
方法大意
应用场景
粗略优缺点
```

它很难严谨写出：

```text
公式推导
实验设置
benchmark 结果
paper 之间的细粒度差异
真实 limitation
可靠 citation-backed conclusion
```

因此这个 lab 的质量上限主要取决于：

```text
research_agent 给 writer_agent 的 evidence packet 有多厚
```

当前 notebook 更像教学 demo：

```text
search snippets / abstracts
  -> summarize
  -> write report
```

不是严肃 research pipeline：

```text
retrieve full paper
  -> parse sections
  -> extract claims/evidence
  -> compare methods/results
  -> write citation-grounded report
```

### Step 6: editor_agent 如何参与修改？

后面如果有一步：

```text
Critique the draft for clarity, structure, and missing citations.
```

executor 可能把它路由到：

```json
{
  "agent": "editor_agent",
  "task": "Review the draft and suggest improvements in clarity, structure, and citation support."
}
```

这时 `history` 已经包含 research output 和 writer draft。

executor 会把所有历史都拼进 context：

```text
Step 1 executed by research_agent:
Key sources found...

Step 2 executed by writer_agent:
The Ensemble Kalman Filter is...

Step 3 executed by writer_agent:
# Ensemble Kalman Filter for Time Series Forecasting
...
```

然后传给 editor：

```text
You are editor_agent.

Here is the context of what has been done so far:
...

Your next task is:
Review the draft and suggest improvements in clarity, structure, and citation support.
```

`editor_agent` 返回的可能是：

```text
The draft is well structured, but it should:
1. Add a clearer distinction between Kalman Filter and Ensemble Kalman Filter.
2. Cite Evensen when introducing EnKF.
3. Add limitations around ensemble size and nonlinear dynamics.
4. Make the applications section more concrete.
```

executor 再 append：

```python
history.append((step, "editor_agent", output))
```

### Step 7: 最终 Markdown 是怎么来的？

最后一步通常是：

```text
Generate a final Markdown document with the complete research report.
```

executor 可能路由给 `writer_agent`：

```json
{
  "agent": "writer_agent",
  "task": "Revise the report using all previous research findings and editor feedback, then output the final report as Markdown."
}
```

这时 writer 能看到：

```text
research findings
summary
draft
editor feedback
```

它输出 Markdown：

```markdown
# Ensemble Kalman Filter for Time Series Forecasting

## Overview
...

## Key Methods
...

## Applications
...

## Limitations
...

## References
...
```

notebook 最后取 `executor_history` 的最后一条：

```python
md = executor_history[-1][-1].strip("`")
display(Markdown(md))
```

这里：

```python
executor_history[-1]
```

是最后一个 step 的 tuple：

```python
(step, agent_name, output)
```

所以：

```python
executor_history[-1][-1]
```

就是最后一个 agent 的输出文本，也就是最终 Markdown report。

### message 和 history 的区别

这个 notebook 里有两种上下文管理。

第一种是每个 agent 内部的 `messages`：

```python
messages = [...]
client.chat.completions.create(messages=messages)
```

这是某一次 LLM call 的 prompt/context。

注意：`writer_agent`、`editor_agent`、`research_agent` 每次被调用时，都会重新创建自己的 `messages`。它们没有长期保存自己的 conversation memory。

第二种是 executor 维护的 `history`：

```python
history = [
    (step, agent_name, output),
    ...
]
```

这是整个 workflow 的主记忆。

agent 之间的通信靠的是：

```text
previous agent output
  -> executor history
  -> context string
  -> enriched_task
  -> next agent input
```

不是：

```text
research_agent 直接 message writer_agent
```

而是：

```text
research_agent
  -> output
  -> executor history
  -> writer_agent input
```

这是一种 manager-mediated communication：

```text
executor 像项目经理。
research_agent 把结果交给 executor。
executor 把结果整理进任务说明。
writer_agent 从任务说明里读到前面的结果。
```

### sub-agent 和 tool 的边界

这个 notebook 里容易混淆三层 callable：

```text
executor 调用 sub-agent
sub-agent 调用 LLM
research_agent 内部的 LLM 调用 tools
```

更精确的边界是：

```text
sub-agent:
普通 Python 函数，由 executor 通过 agent_registry 调用。

tool:
暴露给 LLM 的函数能力，由 LLM 通过 tool calling 选择调用。

history:
executor 维护的 workflow-level memory，用来让下一个 agent 看到前面 agent 的输出。
```

所以：

```text
tool 是给 LLM 调的。
sub-agent 是给 orchestrator/executor 调的。
history 是给 agent 之间传上下文用的。
```

### 这个设计的质量上限和改进方向

这个 lab 的设计很适合教学，因为它把 planning、tool use、multi-agent coordination、reflection 都串起来了。

但如果要变成更可靠的 production research agent，我会优先改这些点：

1. 让 `research_agent` 返回结构化 evidence packet，而不是一段自由文本。

```json
[
  {
    "title": "...",
    "authors": ["..."],
    "url": "...",
    "abstract": "...",
    "key_claims": ["...", "..."],
    "methods": "...",
    "evidence": "...",
    "limitations": "...",
    "relevance_to_topic": "..."
  }
]
```

2. 让 writer 明确只能基于 evidence 写，不要补不存在的细节。

```text
Use only the evidence above.
Cite every technical claim.
If evidence is insufficient, say so.
Do not invent paper details.
```

3. 增加 PDF/full-text reading tools。

```text
search
  -> download PDF
  -> extract text
  -> chunk sections
  -> summarize method / experiment / result / limitation
```

4. 保存完整 trace。

现在 writer 主要看到 research_agent 的 final output。更好的做法是把 tool calls、tool results、source metadata、citation map 都保存下来，方便 eval 和 debug。

5. 给 executor 加 validation / retry。

当前 executor 假设 LLM router 一定返回合法 JSON：

```python
agent_info = json.loads(cleaned_json)
```

生产环境应该加：

```text
JSON schema validation
agent allowlist
retry on invalid JSON
max step limit
error recovery
trace logging
```

### 一句话总结

这个 notebook 里的 multi-agent 通信不是 agent 之间直接聊天，而是 executor 维护一个 `history`，把每个 sub-agent 的输出拼进下一个 sub-agent 的任务上下文。sub-agent 本身是普通 Python 函数，由 `agent_registry[agent_name](enriched_task)` 调用；真正的外部工具只存在于 `research_agent` 内部，由它的 LLM/tool loop 自动选择调用。当前 research agent 主要传递摘要级证据，所以最终 report 的深度也主要受摘要质量限制。
