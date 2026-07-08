---
course: Agentic AI
module: 3
lesson: 6
title: Code Execution
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/a4gs14/code-execution
watched_at: 2026-07-08
reviewed_at:
tags: [tool-use, code-execution, sandbox, reflection, external-feedback, safety]
---

# Code Execution

## TL;DR

Code execution 是一种特殊而强大的 tool：不是给 LLM 准备一堆固定函数，而是让 LLM 写代码，再由外部系统执行代码，把输出或错误信息返回给 LLM。它能显著扩展 agent 能力，尤其适合计算、数据处理、图表、文件转换和复杂推理；但它也带来安全风险，所以真实系统里最好放进 sandbox，并做好权限、日志和回滚。

## Core Ideas

- 如果每个数学能力都做成一个独立 tool，会很快膨胀：加减乘除、平方根、指数、统计函数、绘图函数等等。
- 另一种方式是提供 code execution tool：让 LLM 生成 Python 代码，由系统执行代码并返回结果。
- 常见流程是：LLM 生成代码片段 -> 系统提取代码 -> 执行代码 -> 把 stdout、结果或 error message 回传给 LLM -> LLM 生成最终回答。
- 代码执行失败时，错误日志本身就是 external feedback，可以触发 reflection / revise / retry。
- Python 的 `exec` 可以直接执行字符串代码，但执行任意 LLM-generated code 有安全风险。
- 更稳妥的实践是把代码放进 sandbox，比如 Docker 或 E2B 这类隔离环境。
- Code execution 是很多 LLM 应用重点优化的能力，因为它让模型把“语言推理”连接到“可执行计算”。

## Basic Workflow

```text
User query
  -> LLM writes code
  -> system extracts code
  -> code execution tool runs code
  -> execution output/error returns to LLM
  -> LLM answers or revises code
```

课程用数学题举例：如果用户问 `sqrt(2)`，与其单独写一个 `sqrt` tool，不如让 LLM 写 Python 代码调用 `math.sqrt(2)`。系统执行后得到数值，再让 LLM 把结果组织成自然语言回答。

## Why Code Execution Is Different

普通 tool 的能力边界由开发者预先定义：

```text
add_numbers
subtract_numbers
search_emails
delete_email
```

Code execution tool 的边界更宽：

```text
LLM can write new code for the current task
```

这意味着它不只是在选择已有函数，而是在临时构造一个小程序。这个能力很强，但也更需要边界。

## Extraction Pattern

一种简单做法是要求模型用特殊标签包住代码：

```text
<execute_python>
import math
result = math.sqrt(2)
print(result)
</execute_python>
```

然后系统用 pattern matching 或 regex 提取标签中的代码，交给执行环境运行。这个模式和前面 external feedback 很像：工具输出成为模型下一步回答的新证据。

## Reflection Loop With Errors

如果代码执行报错，不要只返回失败。更有用的是把错误信息作为 feedback 给 LLM：

```text
code attempt
  -> execution error
  -> send traceback/error back to LLM
  -> LLM revises code
  -> run again
```

这相当于把 code execution 和 reflection 结合起来。对数据分析任务尤其重要，因为 SQL/Python/plotting 代码经常需要通过真实运行才能发现问题。

## Safety: Why Sandbox Matters

执行 LLM 写的任意代码有风险。风险不只是“代码跑不通”，还包括：

- 删除或覆盖文件。
- 读取敏感文件或环境变量。
- 访问网络并泄露数据。
- 写数据库或调用高风险 API。
- 占用太多 CPU、内存或磁盘。
- 进入无限循环。

因此，生产系统里最好不要直接在主机环境执行任意代码。更好的做法是放进 sandbox，并限制文件系统、网络、资源、超时和权限。

## Sandbox Options

| Option | Idea | Notes |
| --- | --- | --- |
| Python `exec` | 直接执行字符串代码 | 简单但风险最高，适合本地小实验或受信代码 |
| Docker | 在容器里执行代码 | 隔离文件系统和依赖，适合工程化 |
| E2B / sandbox service | 托管轻量沙箱 | 适合快速给 LLM 应用接入安全执行环境 |

## Design Takeaways

- Code execution 是 tool use 的放大器：一个工具能覆盖很多临时计算和数据处理需求。
- 对可运行任务，execution result 往往比 LLM 自我判断更可靠。
- Error message 是高质量 external feedback，应该被结构化返回给 LLM。
- Code execution loop 必须有 retry limit、timeout、日志和失败退出条件。
- 安全边界要在执行层实现，不要只靠 prompt 要求模型“不要做危险操作”。
- 对数据分析 agent，code execution 适合 Python 计算、pandas 处理、绘图、文件解析和 sanity check。

## Failure Modes

- LLM 生成的代码语法错误或缺少依赖。
- 执行环境没有必要的数据文件、package 或权限。
- 代码输出太长，塞回上下文后干扰模型。
- 只返回 `failed`，没有返回具体 traceback，导致模型无法修正。
- 在非 sandbox 环境执行任意代码，造成文件删除、数据泄露或资源耗尽。
- Retry 没有上限，代码反复失败仍持续执行。

## Implementation Hooks

- Prompt: 要求模型把代码包在明确标签或结构化 tool call 里。
- Parser: 提取代码块，并拒绝多余文本或多个不明确代码块。
- Sandbox: 限制文件系统、网络、环境变量、CPU、内存、磁盘和执行时间。
- Execution result: 返回 stdout、stderr、exit code、exception、artifacts path。
- Reflection: 失败时把 error message 回传给 LLM，让它最多修正 1-2 次。
- Logging: 记录 generated code、execution result、runtime、resource usage、retry count。
- Human review: 对写文件、删文件、发请求、调用业务 API 等副作用操作加确认。

## My Questions

- 数据分析 agent 的 Python execution tool 应该默认允许哪些 package：pandas、numpy、matplotlib、seaborn、plotly？
- 如果用户上传数据文件，sandbox 应该如何挂载文件，才能既可用又不泄露其他目录？
- Code execution 的输出应该返回原始 stdout，还是结构化摘要和 artifact link？
- 失败后最多 retry 几次？什么时候应该停止并向用户解释错误？
- 对生产环境，Docker 和 E2B 这类 sandbox 的成本、延迟和安全边界怎么权衡？

## Related

- [Tool Syntax](03-tool-syntax.md)
- [Tool use](../../concepts/tool-use.md)
- [Code execution tool](../../concepts/code-execution-tool.md)
- [External feedback](../../concepts/external-feedback.md)
- [Reflection pattern](../../concepts/reflection-pattern.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 3 README。
- [x] 新增 code execution tool concept card。
- [x] 补充 glossary 中的 code execution、sandbox、stdout/stderr、traceback。
- [x] 下一节 `MCP` 后，补充 tool ecosystem / protocol 的标准化理解。
- [ ] 未来做数据分析 agent lab 时，设计一个最小 Python execution sandbox。
