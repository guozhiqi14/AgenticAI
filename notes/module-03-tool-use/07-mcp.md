---
course: Agentic AI
module: 3
lesson: 7
title: MCP
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/x7jowg/mcp
watched_at: 2026-07-08
reviewed_at:
tags: [mcp, model-context-protocol, tool-use, client-server, resources, integrations]
---

# MCP

## TL;DR

MCP（Model Context Protocol）是一个让 LLM 应用更容易接入外部 tools 和 data sources 的标准协议。它把过去每个应用都要分别对接 Slack、Google Drive、GitHub、Postgres 等系统的重复工作，变成“应用实现 MCP client，数据源/工具实现 MCP server”的标准化连接。核心价值是把集成成本从 `M applications * N tools` 降到 `M applications + N tools`。

## Core Ideas

- MCP 最初由 Anthropic 提出，现在被很多公司和开发者采用。
- 它的目标是让 LLM 更容易获得 context、resources 和 tools。
- 没有标准协议时，每个应用团队都要自己写 Slack/GitHub/Google Drive/Postgres 的 wrapper。
- 有 MCP 后，应用端只要实现 MCP client，工具/数据源端提供 MCP server，就能复用生态里的集成。
- MCP 的早期设计强调给 LLM fetch data / context；文档里这类数据源常被称为 resources。
- MCP 不只提供静态数据，也可以提供更一般的 tools，让应用执行动作。
- MCP 生态里有两类重要角色：MCP client 和 MCP server。

## The Integration Problem

没有 MCP 时，集成关系像这样：

```text
App 1 -> Slack, GitHub, Google Drive, Postgres
App 2 -> Slack, GitHub, Google Drive, Postgres
App 3 -> Slack, GitHub, Google Drive, Postgres
```

如果有 `M` 个应用、`N` 个工具或数据源，社区总工作量接近：

```text
M * N
```

每个应用都在重复写相似 wrapper。

MCP 的思路是把连接标准化：

```text
MCP clients  <->  MCP protocol  <->  MCP servers
```

这样应用端和工具端各自对接标准协议，总工作量更接近：

```text
M + N
```

## MCP Client And Server

| Role | Meaning | Example |
| --- | --- | --- |
| MCP client | 想使用 tools/resources 的应用 | Claude Desktop、一个自建数据分析 agent |
| MCP server | 提供 tools/resources 的服务包装层 | GitHub MCP server、Slack MCP server、Postgres MCP server |

你自己的 agentic application 未来可能是 MCP client：它需要调用别人已经封装好的 tools/resources。

如果你想把公司内部数据源、指标系统或业务工具开放给其他 agent 使用，也可以做 MCP server。

## Resources And Tools

MCP 里经常会提到 resources。可以先简单理解为：

```text
resource = 可读取的数据或上下文
tool = 可调用的动作或函数
```

例如 GitHub MCP server 可能提供：

- 读取 repo 的 `README.md` 文件。
- 列出最新 pull requests。
- 获取 issue、commit、file content 等上下文。

这些结果会被放回 LLM context，让模型基于真实外部信息回答。

## Example: GitHub MCP Server

课程演示了一个 MCP client 连接 GitHub MCP server 的例子：

```text
User asks:
  summarize README.md from a GitHub repo

MCP client asks GitHub MCP server:
  get file README.md from repo

GitHub MCP server returns:
  file content

LLM:
  summarizes the markdown
```

另一个例子是用户问最新 pull requests。应用会通过 GitHub MCP server 调用 list pull requests 这类工具，拿到结构化结果后，再让 LLM 生成自然语言总结。

## Design Takeaways

- MCP 是 tool use 生态化的关键：不必每个团队都从零封装每个工具。
- 对应用开发者来说，MCP client 能快速接入已有工具和数据源。
- 对平台/数据团队来说，MCP server 是把内部能力标准化暴露给 agent 的方式。
- MCP resources 很适合业务知识库、文档、代码仓库、数据目录、指标口径等 context source。
- MCP tools 可以封装搜索、查询、创建、更新等动作，但仍然需要权限、日志和 human review。
- 标准协议降低了集成成本，但不自动解决安全、权限、质量和 eval 问题。

## Failure Modes

- 把 MCP 当成“安全保证”，忽略 server 端权限控制。
- 暴露过多 resources，导致 LLM context 变长、变杂、变贵。
- MCP server 返回结果格式不稳定，client 难以可靠使用。
- Tool/resource 描述不清，LLM 不知道何时调用哪个能力。
- 没有记录 MCP request/response trace，debug tool use 失败很困难。
- 直接接入第三方 MCP server，但没有评估数据质量、权限和合规边界。

## Implementation Hooks

- Client side: 你的 agent app 可以作为 MCP client，连接 GitHub、Postgres、Slack 等 MCP servers。
- Server side: 可以把内部指标系统、数据目录、业务 wiki、实验平台封装成 MCP server。
- Resource design: 给 LLM 提供稳定、短小、结构化的 context，而不是一次塞全量文档。
- Tool design: 为每个 action 定义清楚输入、输出、权限和错误格式。
- Observability: 记录 MCP server、resource/tool name、arguments、response size、latency、error。
- Evals: 评估模型是否在正确任务里调用正确 MCP resource/tool。

## My Questions

- 对数据分析 agent，最适合先做 MCP server 的内部能力是什么：schema lookup、metric definition、SQL execution、实验平台还是业务 wiki？
- MCP resource 返回原文、摘要、结构化 JSON，哪个更适合 LLM 稳定使用？
- 公司内部 MCP server 应该如何做权限：按用户身份、按工具、按数据表，还是按 action？
- 如果接入多个 MCP servers，如何避免 tool/resource 过多导致模型选择困难？
- MCP request/response 应该如何纳入 trace 和 eval？

## Related

- [What are tools?](01-what-are-tools.md)
- [Tool Syntax](03-tool-syntax.md)
- [Code Execution](06-code-execution.md)
- [Tool use](../../concepts/tool-use.md)
- [Model Context Protocol](../../concepts/model-context-protocol.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 更新 Module 3 README。
- [x] 新增 MCP concept card。
- [x] 补充 glossary 中的 MCP、MCP client、MCP server、resource。
- [ ] 完成 Module 3 quiz 后，回补 quiz 里的误区和自测题。
- [ ] Module 4 evals 开始后，把 tool selection / MCP call selection 纳入 eval 思路。
