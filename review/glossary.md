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

## Deterministic Workflow

步骤顺序预先固定的 workflow。LLM 仍然可以在这些固定步骤里生成文本或结构化输出。

## Reflection

系统在最终输出前，回看并改进自己的 output 或 plan。

## Eval

用来判断 agentic workflow 是否做对事情的测试或评审过程，既包括 intermediate action choices，也包括 final output quality。
