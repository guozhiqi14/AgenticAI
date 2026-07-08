# Concept: Model Context Protocol

## Definition

MCP（Model Context Protocol）是一个让 LLM 应用标准化接入 tools 和 data sources 的协议。它定义了应用如何作为 MCP client 去连接 MCP server，并从 server 获取 resources 或调用 tools。

一句话理解：

```text
MCP = 给 agent 工具生态用的标准插座
```

## Why It Matters

没有 MCP 时，每个 agent 应用都要单独集成每个外部系统：

```text
M applications * N tools/data sources
```

有 MCP 后，应用端实现 MCP client，工具/数据源端实现 MCP server：

```text
M applications + N tools/data sources
```

这让社区和公司内部都能复用工具集成，而不是反复写 wrapper。

## Core Roles

| Role | Description | Example |
| --- | --- | --- |
| MCP client | 想使用工具/上下文的 LLM 应用 | Claude Desktop、自建数据分析 agent |
| MCP server | 提供工具/上下文的服务包装层 | GitHub MCP server、Postgres MCP server |
| Resource | 可读取的数据或上下文 | README、issue、表 schema、指标定义 |
| Tool | 可调用的函数或动作 | list pull requests、query database、send email |

## Basic Shape

```text
LLM app / MCP client
  -> asks MCP server for resource or tool call
  -> MCP server talks to real external system
  -> returns result
  -> result goes back into LLM context
```

对用户来说，看起来像 agent “会用 GitHub / Slack / Postgres”。实际上是 MCP client 通过 MCP server 调用了这些系统。

## Data Analyst Mental Model

可以把 MCP server 理解成一个标准化的数据/工具网关：

```text
业务数据表
指标口径文档
实验平台
SQL 执行服务
知识库
```

这些内部能力如果被封装成 MCP server，数据分析 agent 就可以用统一方式查 schema、取指标定义、运行查询、读业务文档。

## Design Questions

- 这个 MCP server 提供的是 resource、tool，还是两者都有？
- Resource 返回多大、多结构化、多可信？
- Tool 是否有副作用？是否需要 human approval？
- 权限应该在 MCP server、底层 API，还是两者都做？
- MCP response 是否有稳定 schema？
- 多个 MCP servers 同时存在时，LLM 如何选择正确 server/tool？
- MCP 调用如何记录 trace、latency、cost、error？

## Failure Modes

- MCP server 暴露过宽，导致越权访问或数据泄露。
- Resource 返回太长，污染 context 或增加成本。
- Tool 描述不清，模型误调用或漏调用。
- MCP server 错误信息不清，LLM 无法修正下一步。
- 没有 eval，无法判断 MCP 接入后是否真的改善最终任务。
- 把 MCP 当作架构银弹，忽略权限、质量、观测和产品体验。

## Practical Rule

对内部数据分析 agent，优先考虑把“稳定、高频、跨场景”的能力做成 MCP server：

1. schema/table metadata lookup
2. metric definition retrieval
3. business wiki/document search
4. SQL execution with permission control
5. experiment/AB test metadata lookup

不要一开始把所有系统都暴露给 LLM。先从少量高价值 resources/tools 做起，并配套 eval 和 trace。

## Related Notes

- [MCP](../notes/module-03-tool-use/07-mcp.md)
- [Tool use](tool-use.md)
- [Tool schema](tool-schema.md)
- [Code execution tool](code-execution-tool.md)
- [Agentic evals](agentic-evals.md)
