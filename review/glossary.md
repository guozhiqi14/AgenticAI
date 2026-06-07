# Glossary

## Agentic

一个系统具有的性质，描述它能在多大程度上做决策、选择步骤、使用工具或调整 workflow。

## Autonomy

系统在多大程度上可以决定下一步做什么，而不是每一步都由工程师预先写死。

## Tool Use

LLM-driven workflow 调用外部软件、API、代码、搜索、文件读取器或其他函数，用来执行动作或获取信息。

## Web Search

用关键词搜索互联网或某个资料库，返回候选结果列表。它主要用于发现可能相关的信息来源。

## Web Fetch

给定一个具体 URL 或文件地址，把该来源的内容读取回来。它主要用于让系统真正阅读、摘要、分析或引用某个来源。

## Parallelism

把多个互不依赖的子任务同时执行，比如同时生成多个搜索 query、同时 fetch 多个网页，或同时生成多个候选方案。

## Modularity

把 workflow 拆成可替换组件，每个组件有清晰的输入和输出。这样可以替换模型、搜索引擎、工具或 parser，而不用重写整个系统。

## Human-in-the-loop

把人工审核、批准或干预放进 workflow。常见用法是让 agent 起草、提取或准备动作，但在真正发送邮件、更新数据库、付款、退货等高风险动作前由人确认。

## Computer Use Agent

能够操作浏览器或桌面软件的 agent。它需要读取页面状态、点击、输入、等待加载并调整策略。相比稳定 API，computer use 通常更难可靠。

## SOP

Standard Operating Procedure，标准作业流程。已有 SOP 的任务通常更适合先做 agentic workflow，因为步骤更容易拆解、实现和评估。

## Task Decomposition

把复杂任务拆成一串 discrete steps，并判断每一步可以由 LLM、API、function call、code execution、retrieval、database query 或 human review 完成。

## Building Block

构建 agentic workflow 的可用能力单元，例如 LLM、multimodal model、specialized AI model、API、RAG、database tool、code execution 或 human review。

## RAG

Retrieval-Augmented Generation。先从外部知识库或文本集合中检索相关内容，再把这些内容交给 LLM 生成答案。

## Deterministic Workflow

步骤顺序预先固定的 workflow。LLM 仍然可以在这些固定步骤里生成文本或结构化输出。

## Reflection

系统在最终输出前，回看并改进自己的 output 或 plan。

## Planning

让 LLM 在运行时决定完成任务需要哪些步骤、调用哪些工具、以及这些步骤的顺序。Planning 通常更灵活，但也更难控制和评估。

## Multi-Agent Collaboration

让多个 agent 扮演不同角色并协作完成任务。每个 agent 通常是 LLM 加不同 role prompt，例如 researcher、writer、critic、tester、editor。

## Design Pattern

组合 building blocks 的常见 workflow 结构，例如 reflection、tool use、planning 和 multi-agent collaboration。

## Eval

用来判断 agentic workflow 是否做对事情的测试或评审过程，既包括 intermediate action choices，也包括 final output quality。

## Objective Eval

判断标准明确、可以用代码自动检查的 eval，比如是否提到禁用词、是否包含必填字段、JSON 是否符合 schema。

## LLM-as-judge

用另一个 LLM 来评估输出质量的技术，适合更主观的判断，比如文章质量、回复礼貌程度、答案完整性。需要清晰 rubric，不能盲信简单分数。

## End-to-End Eval

评估整个 agentic workflow 的最终输出是否满足用户或业务目标。

## Component-Level Eval

评估 workflow 中某个单独步骤的质量，比如字段提取、query generation、tool selection、retrieval 或 draft response。

## Trace

Agentic workflow 的中间执行记录，包括每一步输入输出、工具调用、模型判断、错误、耗时和成本。

## Error Analysis

系统化阅读 outputs 和 traces，找出 recurring failure patterns，并把这些失败转成 eval 和改进任务。
