# Concept: Planning Pattern

## Definition

Planning pattern 是让 LLM 在 runtime 根据用户请求、可用工具和上下文生成 step-by-step plan，再由 executor 按计划逐步执行的 agentic design pattern。

它和普通 task decomposition 的区别在于：task decomposition 可以是 developer 在设计时做的；planning 是模型在运行时为当前请求动态做的。

## Why It Matters

很多真实任务没有固定步骤。用户可能问库存、退货、邮件处理、代码实现、数据分析或市场研究；这些任务可能使用同一组 tools，但调用顺序不同。

Planning 把系统从“固定流程自动化”推向“动态流程编排”。这能覆盖更宽的请求空间，但也让系统更难预测、debug 和评估。

## Basic Shape

```text
User request
  -> planner LLM sees tools and constraints
  -> planner outputs structured plan
  -> executor runs step 1
  -> step 1 output becomes context
  -> executor runs step 2
  -> ...
  -> final response/action
```

Plan 最好不是自由散文，而是结构化 artifact。这样系统才能记录、检查、重试和评估。

## Structured Plan Format

可执行 plan 应该尽量用 JSON 或 XML 表达，而不是只写成 plain text。

一个常见 plan schema 至少包含：

- `steps`: step list。
- `step`: step number or id。
- `description`: 这一步要完成什么。
- `tool`: 这一步建议使用哪个 tool。
- `arguments`: 传给 tool 的参数。
- `depends_on`: 可选，说明依赖哪些前置 step。
- `success_criteria`: 可选，说明这一步怎样算成功。

JSON/XML 的价值是让 downstream code 能 parse 和 validate。Markdown 或普通英文适合给人看，但不适合作为高可靠 executor 的唯一输入。

## Code-As-Plan

当任务天然可以用代码完成时，planner 不一定输出 JSON/XML plan，而可以直接输出可执行代码。代码里的函数调用、变量、中间结果、循环和条件分支共同表达 plan。

这种方式适合：

- CSV/Excel/JSON 等数据处理。
- pandas / Python 分析。
- 计算、排序、过滤、聚合、去重。
- 文件转换和 artifact 生成。

它的优势是减少 tool explosion：不需要为每一种数据操作都封装一个独立工具。风险是 runtime 行为更难预测，所以必须配套 sandbox、timeout、resource limits、file/network policy 和 execution trace。

## Planning vs Hard-Coded Workflow

| Choice | Who Decides Step Order | Best For | Risk |
| --- | --- | --- | --- |
| Hard-coded workflow | Developer | 稳定、重复、高风险、合规强的流程 | 覆盖不了变化大的请求 |
| Planning workflow | LLM at runtime | 开放、多工具、动态步骤的任务 | 难控制、难 eval、可能误规划 |
| Code-as-plan | LLM writes executable code | 数据处理、计算、agentic coding | 需要 sandbox，代码可能越权或逻辑错误 |

## When To Use

- 用户请求类型很多，无法穷举固定流程。
- 可用 tools 很多，需要根据上下文决定调用顺序。
- 每一步依赖前一步结果。
- 任务可以拆成 checklist 式步骤。
- 中间 plan 和 trace 可以被保存、审核和评估。

## When To Avoid

- 步骤顺序已经稳定，而且安全/合规要求高。
- 错误工具调用会造成不可逆副作用。
- 缺少 plan logging、guardrails 或 eval。
- 任务很简单，direct generation 或 fixed tool call 已经够用。
- 主要失败模式不是步骤选择，而是某个固定 component 的质量问题。

## Implementation Hooks

- `planner_prompt`: 描述用户目标、可用工具、禁止动作、输出格式。
- `plan_schema`: 每步包含 goal、inputs、allowed tools、expected output、success criteria。
- `parser_validator`: 检查 JSON/XML 格式、必填字段、tool allowlist 和 arguments schema。
- `executor`: 逐步执行 plan，并把每步 output 写回上下文。
- `code_runner`: 当使用 code-as-plan 时，负责在 sandbox 中运行生成代码。
- `trace`: 保存 plan、step input/output、tool calls、latency、cost、errors。
- `guardrails`: 对外部副作用、高成本调用和敏感数据访问做限制。
- `plan_eval`: 检查完整性、顺序、工具选择、风险和冗余。
- `end_to_end_eval`: 验证最终结果是否满足用户目标。

## Failure Modes

- Missing step: plan 漏掉必要检查或信息来源。
- Wrong order: step 顺序导致后续上下文错误。
- Tool misuse: 选错工具或传错参数。
- Invalid structure: plan 不是合法 JSON/XML，或者缺少必填字段。
- Unsafe code: code-as-plan 访问敏感文件、网络或外部系统。
- Over-planning: 步骤太多，增加 latency/cost。
- Unsafe action: plan 包含删除、发送、退款、写数据库等高风险动作。
- Context loss: executor 没有正确使用前一步输出。
- Invisible failure: 没有保存 plan/trace，出错后无法复盘。

## Practical Rule

使用 planning 前先问：

> 这个任务真正需要 runtime 决定步骤吗？

如果答案是否定的，优先用更简单的 fixed workflow。只有当请求空间足够宽、工具组合足够动态，并且你有 trace/eval/guardrails 时，planning 才值得引入。

## Related Notes

- [Planning workflows](../notes/module-05-patterns-for-highly-autonomous-agents/01-planning-workflows.md)
- [Agentic design patterns](agentic-design-patterns.md)
- [Autonomy spectrum](autonomy-spectrum.md)
- [Task decomposition](task-decomposition.md)
- [Tool use](tool-use.md)
- [Code execution tool](code-execution-tool.md)
- [Agentic evals](agentic-evals.md)
