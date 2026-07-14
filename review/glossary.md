# Glossary

## Agentic

一个系统具有的性质，描述它能在多大程度上做决策、选择步骤、使用工具或调整 workflow。

## Autonomy

系统在多大程度上可以决定下一步做什么，而不是每一步都由工程师预先写死。

## Tool Use

LLM-driven workflow 让模型请求调用外部软件、API、代码、搜索、文件读取器或其他函数，用来执行动作或获取信息。工具在 LLM 外部执行，结果再返回给 LLM。

## Tool

开发者提供给 LLM 使用的函数或外部能力，例如 `getCurrentTime`、web search、SQL query、calendar API、calculator。LLM 可以根据任务判断是否请求调用它。

## Tool Call

LLM 请求调用某个工具，并提供需要的参数。真正执行工具的是外部系统，不是 LLM 自己。

## Tool Call Request

LLM 生成的结构化请求，表示它希望系统调用某个工具。请求通常包含 tool name 和 arguments。

## Tool Arguments

工具调用时传入的参数，例如 time zone、search query、SQL、customer id。参数设计越清楚，LLM 越容易稳定调用工具。

## Tool Result

工具执行后返回给 LLM 的结果。LLM 会把这个结果作为新上下文，再生成答案或决定下一步工具调用。

## Tool Registry

应用层维护的一张“工具名称 -> 实际函数”的映射表。系统根据 LLM 的 tool call request 找到对应函数并执行。

## Function Calling

一种实现 tool use 的方式：开发者把函数名称、描述、参数 schema 暴露给模型，模型输出一个结构化 tool call，请系统执行函数。

## Tool Syntax

把工具暴露给 LLM 的结构化表达方式。它通常包含工具列表、tool schema、messages、model，以及控制 tool-call loop 的参数。

## Tool Schema

给 LLM 看的工具接口说明，通常包含 tool name、description、parameters、required fields 和参数类型。好的 tool schema 能帮助模型正确选择工具并填写参数。

## JSON Schema

一种结构化描述数据形状的格式。Tool calling 中常用它描述工具参数，例如参数名、类型、必填字段和可选枚举值。

## Docstring

Python 函数或类下面的说明文字。某些 tool-calling library 会用 docstring 自动生成 tool description，所以它不只是给人看的注释，也可能影响 LLM 如何使用工具。

## Max Turns

Tool-call loop 的最大轮数限制。它防止模型连续请求工具调用而停不下来，是控制成本、延迟和异常循环的安全阀。

## Code Execution

让 LLM 生成代码，并由外部系统执行代码的能力。执行结果、错误日志或生成文件会返回给 LLM，用来回答用户或继续修正。

## Code-As-Plan

Planning pattern 的一种形式：让 LLM 用可执行代码表达 plan。代码里的函数调用、变量、中间结果、循环和条件分支共同代表计划步骤。它适合数据处理、计算、文件转换和 agentic coding，但必须放在 sandbox 中执行。

## Tool Explosion

为了覆盖越来越多用户请求，不断新增细粒度工具，导致工具数量膨胀、选择困难、维护成本上升和 edge cases 增多。Code execution 有时可以缓解这种问题。

## Sandbox

隔离执行环境，用来限制代码能访问的文件、网络、环境变量、CPU、内存、磁盘和执行时间。Code execution tool 最好在 sandbox 中运行。

## Python `exec`

Python 内置函数，可以执行字符串形式的 Python 代码。它很强大，但直接执行 LLM 生成的任意代码有安全风险。

## stdout / stderr

程序执行时的标准输出和标准错误。stdout 通常放正常结果，stderr 通常放错误或诊断信息。Code execution tool 会把这些结果回传给 LLM。

## Traceback

Python 报错时展示的调用链和错误位置。把 traceback 反馈给 LLM，常常能帮助它 revise code 并重试。

## Docker

常见容器技术，可以把代码放在隔离环境中运行，降低影响主机文件系统和依赖环境的风险。

## E2B

一种面向 AI 应用的轻量 sandbox 服务，可以用来运行 LLM 生成的代码，并隔离执行环境。

## MCP / Model Context Protocol

一种让 LLM 应用标准化接入 tools 和 data sources 的协议。它让应用作为 MCP client 连接 MCP server，从而获取 resources 或调用 tools。

## MCP Client

想使用 MCP tools/resources 的 LLM 应用。比如桌面 AI 应用、自建数据分析 agent 或其他 agentic application。

## MCP Server

提供 MCP tools/resources 的服务包装层。它通常连接真实外部系统，比如 GitHub、Slack、Google Drive、Postgres、业务知识库或内部指标平台。

## Resource

MCP 语境中，resource 通常指可读取的数据或上下文，例如文件内容、repo README、issue、表 schema、指标定义或业务文档。

## HTTP

客户端和服务器之间通信的一套规则。核心模型是 request -> response：客户端发送请求，服务器处理后返回结果。浏览器、手机 App、Python 脚本、后端服务和 LLM tool 背后的函数都可以发 HTTP request。

## URL

Uniform Resource Locator，资源地址。它告诉客户端要访问哪个服务器、哪个路径或资源，例如 `https://api.example.com/emails/3`。

## HTTP Method

HTTP request 中表示动作的字段。常见 method 包括 `GET` 读取、`POST` 创建或提交、`PATCH` 修改一部分、`DELETE` 删除。

## HTTP Request

客户端发给服务器的请求，通常包含 method、URL/path、headers 和 body。比如 Python tool 可以用 HTTP request 去调用后端 API。

## HTTP Response

服务器返回给客户端的结果，通常包含 status code、headers 和 body。Agent tool 会把 response 里的关键结果再交回 LLM。

## REST Route

后端暴露出来的一个资源操作入口，通常由 HTTP method + path 组成，例如 `GET /emails`、`POST /send`、`DELETE /emails/{email_id}`。课程里的 email tools 可以理解为 Python 函数包装了一组 REST routes。

## Web Search

用关键词搜索互联网或某个资料库，返回候选结果列表。它主要用于发现可能相关的信息来源。

## Web Fetch

给定一个具体 URL 或文件地址，把该来源的内容读取回来。它主要用于让系统真正阅读、摘要、分析或引用某个来源。

## Parallelism

把多个互不依赖的子任务同时执行，比如同时生成多个搜索 query、同时 fetch 多个网页，或同时生成多个候选方案。

## Latency

用户或系统等待 workflow 完成的时间。Agentic workflow 的 latency 通常来自多个 component 的累计耗时，例如 LLM call、web search、web fetch、database query、code execution 和 final generation。

## Cost Optimization

降低 workflow 每次运行成本的过程。常见成本来源包括 LLM input/output tokens、外部 API call、compute/server runtime、warehouse query 和存储/带宽。

## Per-Step Benchmark

把 workflow 拆成多个 step，分别记录每一步的 latency、cost 和质量指标。它用于判断真正值得优化的 component，而不是凭感觉优化。

## Critical Path

决定整体 latency 的最长依赖路径。如果两个步骤可以并行，整体等待时间取决于较慢的那条路径；如果步骤必须串行，它们的耗时会累加。

## Build / Analyze Loop

构建 agentic workflow 时在“动手改系统”和“分析系统表现”之间来回切换的开发循环。Build 包括写代码、调 prompt、换模型、改工具；analyze 包括读 outputs/traces、做 eval、error analysis 和 component attribution。

## Custom Eval

为某个具体 agentic workflow、业务场景或 failure mode 定制的 eval。通用 monitoring 工具可以记录 traces、latency、cost，但 custom eval 才定义这个应用里什么算好、什么算错。

## Observability

让系统运行过程可观察的能力，包括 logging、traces、metrics、latency、cost、errors 和 intermediate outputs。Observability 帮助分析问题，但它本身不等于 eval。

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

## Planning Pattern

让 LLM 在 runtime 根据用户请求、工具集合和上下文生成 step-by-step plan，再由 executor 按计划逐步执行的 agentic design pattern。它适合开放、多工具、步骤顺序变化大的任务，但需要更强的 trace、guardrails 和 eval。

## Plan

Planning workflow 中由模型生成的中间控制结构，通常包含多个 step、每步目标、需要的输入、可能使用的工具和成功标准。Plan 应该被保存和评估，而不是隐藏在模型内部。

## Plan Schema

描述 plan 应该如何结构化输出的 contract，例如 JSON/XML 中的 `steps`、`description`、`tool`、`arguments`、`depends_on` 和 `success_criteria`。Plan schema 让 downstream code 能 parse、validate 和 execute。

## Structured Output

让 LLM 按固定格式输出内容，例如 JSON、XML 或符合某个 schema 的对象。Structured output 的价值是减少解析歧义，让程序能稳定读取模型输出。

## Parser / Validator

Parser 把模型输出解析成程序可用的数据结构；validator 检查字段、类型、tool name、arguments 和安全约束是否符合预期。在 planning workflow 中，它位于 planner 和 executor 之间。

## Planner

负责生成 plan 的 LLM 或 agent component。Planner 的输入通常包括用户请求、可用工具、约束和上下文；输出应尽量结构化，方便后续执行和检查。

## Executor

负责按 plan 执行每个 step 的 component。Executor 会调用工具、读取上一步结果、记录 trace，并把中间结果传给后续步骤或最终回答。

## Reflection

系统在最终输出前，回看并改进自己的 output 或 plan。常见流程是先生成 draft，再 critique/reflect，最后 revise。加入外部反馈如 error logs、test results 或 user feedback 时，reflection 通常更有效。

## External Feedback

来自 LLM 自身之外的新信息，例如代码执行结果、错误日志、单元测试结果、用户反馈、检索到的证据或工具返回值。External feedback 可以显著增强 reflection。

## Feedback Signal

能告诉系统“哪里做得不对或还可以改进”的信息。好的 feedback signal 应该具体、可信、可行动，例如 `JSON invalid`、`contains competitor name X`、`word count is 182 but limit is 150`。

## Pattern Matching

用规则在文本中查找特定模式，比如检查是否出现某些禁用词、竞品名、邮箱、手机号或格式片段。它不一定智能，但对确定性规则很可靠。

## Regular Expression / Regex

一种用于文本匹配的规则语言。可以把它理解成“更强的字符串查找”，常用于识别特定格式或关键词变体。

## Trusted Source

在 fact-checking 中被系统认为可信的信息来源，例如官方文档、业务 wiki、数据库元信息、人工维护的规则表或经过验证的数据源。

## Direct Generation

直接给 LLM 一个 instruction，让它一次性生成最终答案，不加入后续 critique、reflection 或 revision 步骤。它通常是评估 reflection workflow 的 baseline。

## Zero-Shot Prompting

Prompt 中不提供示例，只给任务说明，让模型直接完成任务。

## One-Shot / Few-Shot Prompting

在 prompt 中提供一个或多个 input-output 示例，让模型模仿示例格式或行为完成新任务。

在修复 LLM component 时，few-shot examples 常用于明确边界、输出格式和容易混淆的判断标准。

## Hyperparameter

组件运行时可调、但不是模型自动学习出来的设置。例如 web search 的 result count/date range、RAG retrieval 的 similarity threshold/top-k/chunk size、ML detector 的 confidence threshold。

## Model Selection

为 workflow 的某个 step 选择更合适的模型。选择标准不只是整体 benchmark，还包括 instruction following、reasoning、extraction accuracy、tool use reliability、long-context handling、latency 和 cost。

## Instruction Following

模型遵守具体任务说明、格式要求和限制条件的能力。PII redaction、结构化输出、不要泄露敏感字段、严格按 JSON schema 返回，都是 instruction following 的典型考验。

## Fine-Tuning

用特定任务数据继续训练模型，让它更适合稳定、重复、分布清楚的任务。Fine-tuning 通常需要足够训练数据、baseline、eval set 和人工抽查，不适合作为发现问题后的第一反应。

## Reflection Criteria

Reflection prompt 中明确要求检查的标准，例如格式正确性、完整性、事实准确性、语气、可读性、负面含义、是否容易发音等。

## Ground Truth

用于 eval 的正确答案或参考答案。比如 SQL agent 问“2025 年 5 月卖出多少件商品”，ground truth 就是人工或可信程序确认过的正确数值。

## Per-Example Ground Truth

每个 eval example 都有自己的正确答案或 reference label。例如每张 invoice 的 due date 不同，每个 research topic 的 gold standard talking points 也不同。

## Rubric

用于主观评估的一组明确评分标准。好的 rubric 会把“质量好不好”拆成更具体的维度，例如图表是否有标题、坐标轴标签是否清楚、图表类型是否合适。

## Binary Criteria

只有 0/1 两种结果的评分标准。相比直接让 LLM 打 1-5 分，多个 binary criteria 通常更稳定，也更容易解释。

## Position Bias

LLM-as-judge 在比较两个或多个选项时，可能偏向某个展示位置，比如总是更喜欢第一个选项，而不是真正根据质量判断。

## Multi-Agent Collaboration

让多个 agent 扮演不同角色并协作完成任务。每个 agent 通常是 LLM 加不同 role prompt，例如 researcher、writer、critic、tester、editor。

## Design Pattern

组合 building blocks 的常见 workflow 结构，例如 reflection、tool use、planning 和 multi-agent collaboration。

## Eval

用来判断 agentic workflow 是否做对事情的测试或评审过程，既包括 intermediate action choices，也包括 final output quality。

## Eval Set

用于重复评估 workflow 的一组输入样例、期望标准和评估方法。第一版 eval set 可以很小，比如 10 到 20 个样例，之后随着错误分析持续扩充。

## Objective Eval

判断标准明确、可以用代码自动检查的 eval，比如是否提到禁用词、是否包含必填字段、JSON 是否符合 schema。

## LLM-as-judge

用另一个 LLM 来评估输出质量的技术，适合更主观的判断，比如文章质量、回复礼貌程度、答案完整性。需要清晰 rubric，不能盲信简单分数。

对于图表、报告等主观任务，rubric-based single-output scoring 通常比直接问 LLM “A 和 B 哪个更好”更稳定。

## End-to-End Eval

评估整个 agentic workflow 的最终输出是否满足用户或业务目标。

它从 user input 一直评到 final output，不先拆开中间步骤。适合判断用户最终看到的结果是否变好，但不一定能定位是哪一个 component 出错。

## Component-Level Eval

评估 workflow 中某个单独步骤的质量，比如字段提取、query generation、tool selection、retrieval 或 draft response。

它适合在 error analysis 已经指出某个 component 是瓶颈之后使用。局部 eval 的优点是更快、更便宜、信号更清楚；局部优化后仍然需要 end-to-end eval 确认整体质量变好。

## Gold Standard Resources

在 retrieval 或 web search eval 中，由专家标注的“这个 query 应该找到的权威来源集合”。可以用来评估搜索结果是否覆盖关键来源。

## Precision / Recall / F1

信息检索常用指标。Precision 关注返回结果里有多少是对的，recall 关注该找的结果找回了多少，F1 是 precision 和 recall 的综合平衡指标。

## Trace

Agentic workflow 的中间执行记录，包括每一步输入输出、工具调用、模型判断、错误、耗时和成本。

## Span

Trace 中某个单独步骤或组件的输出记录。例如 search terms、search results、selected sources、SQL draft、tool result 或 chart spec。

## Error Analysis

系统化阅读 outputs 和 traces，找出 recurring failure patterns，并把这些失败转成 eval 和改进任务。

在复杂 agentic workflow 中，error analysis 还要判断错误最早从哪个 component 开始，并避免把上游坏输入导致的问题错误归因给下游。

## Component Attribution

在 error analysis 中，把 final output 的失败归因到具体 workflow component 的过程。例如 invoice due date 错误可能归因到 PDF-to-text，也可能归因到 LLM extraction。

## Non-Mutually Exclusive Errors

错误类别不互斥。一个失败样例可能同时暴露 query generation、database data quality 和 final drafting 问题，所以错误比例可能加起来超过 100%。
