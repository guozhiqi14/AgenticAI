# Concept: Code Execution Tool

## Definition

Code execution tool 是一种允许 LLM 生成代码，并由外部系统执行代码的工具。执行结果、错误日志或生成的 artifact 会再返回给 LLM，用来回答用户或继续修正。

它和普通工具的区别是：普通工具通常是开发者预先写好的固定函数；code execution tool 让模型可以为当前任务临时写一个小程序。

## Why It Matters

很多任务如果为每个能力都写一个专门工具，会很快变得不可维护。比如数学计算、数据清洗、pandas 聚合、画图、文件解析、格式转换，都可能需要很多函数。

Code execution tool 把这些能力统一成：

```text
LLM writes code
system runs code
LLM uses output
```

对数据分析 agent 来说，这非常关键，因为真实分析经常需要运行 Python 或 SQL 才能得到可靠结果。

## Basic Shape

```text
User task
  -> LLM generates code
  -> sandbox executes code
  -> stdout/stderr/artifacts return
  -> LLM summarizes or revises
```

如果执行失败，error message 可以作为 external feedback：

```text
error -> reflection -> revised code -> rerun
```

## Best Fits

- 数学计算和公式推导验证。
- pandas 数据清洗、聚合、join、sanity check。
- 图表生成和文件 artifact 生成。
- JSON/CSV/Excel/PDF 等文件解析。
- 批量文本转换、格式校验。
- 写小脚本验证 LLM 的推理结果。

## Safety Boundary

Code execution 最大的设计问题不是“能不能跑”，而是“在哪里跑、能访问什么、能改什么”。

最低限度要考虑：

- 文件系统隔离。
- 禁止或限制网络访问。
- 隐藏敏感环境变量。
- CPU / memory / disk / time limit。
- 只允许访问任务需要的输入文件。
- 对写文件、删文件、外部 API 调用加权限控制。

Prompt 不是安全边界。真正的安全边界必须在 sandbox 和执行层实现。

## Design Questions

- 这个任务真的需要 code execution，还是固定 tool 更可控？
- 执行环境里预装哪些 package？
- 输入文件如何挂载给 sandbox？
- 输出是 stdout、文件 artifact，还是结构化 JSON？
- 失败时返回完整 traceback，还是摘要后的错误信息？
- 是否允许网络访问？是否允许写文件？
- Retry limit、timeout 和 resource limit 怎么设？
- 哪些 code execution 需要 human review？

## Failure Modes

- 代码执行环境和模型假设不一致，例如缺 package、缺文件、路径错误。
- 模型生成危险代码，例如删除文件或读取敏感路径。
- stdout 太长，导致上下文污染。
- 只把错误吞掉，不把 traceback 返回给模型。
- 没有 sandbox，导致数据泄露或文件系统损坏。
- 没有 retry / timeout 上限，导致成本和延迟失控。

## Related Notes

- [Code Execution](../notes/module-03-tool-use/06-code-execution.md)
- [Tool use](tool-use.md)
- [External feedback](external-feedback.md)
- [Reflection pattern](reflection-pattern.md)
- [Agentic evals](agentic-evals.md)
