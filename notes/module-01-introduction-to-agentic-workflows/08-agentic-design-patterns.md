---
course: Agentic AI
module: 1
lesson: 8
title: Agentic Design Patterns
status: draft
source_url: https://learn.deeplearning.ai/courses/agentic-ai/lesson/rm9bg7/agentic-design-patterns
watched_at: 2026-06-07
reviewed_at:
tags: [design-patterns, reflection, tool-use, planning, multi-agent]
---

# Agentic Design Patterns

## TL;DR

Agentic workflows 可以看成把一组 building blocks 组合成复杂 workflow 的方式。这节课总结了四个核心设计模式：reflection、tool use、planning、multi-agent collaboration。它们分别对应“自我改进”“调用外部能力”“动态决定步骤”和“多个角色协作”。

## Core Ideas

- Design pattern 是组合 building blocks 的常见方式，不是某个单一工具或模型。
- Reflection 让模型检查自己的输出，或者结合外部反馈继续迭代。
- Tool use 让 LLM 调用函数、API、代码执行、搜索、数据库等外部能力。
- Planning 让 LLM 自己决定完成任务需要哪些步骤和顺序。
- Multi-agent collaboration 让多个带有不同角色提示的 agent 协作完成复杂任务。
- 这些模式能提高能力，但 autonomy 和复杂度越高，控制和 eval 也越重要。

## The Four Patterns

| Pattern | Core Move | Example | Main Risk |
| --- | --- | --- | --- |
| Reflection | 让模型检查、批评、修正自己的输出 | 写代码后让模型检查 correctness、style、efficiency，再修复 bug | 不能保证总是变好，可能循环或自信地错 |
| Tool use | 让模型调用外部函数或工具 | Web search、code execution、database、email、calendar | 工具选择错误、外部副作用、成本和安全问题 |
| Planning | 让模型决定行动步骤和顺序 | 根据任务自动选择 pose detection、image generation、image-to-text、text-to-speech 等 API | 更难控制，更实验性 |
| Multi-agent collaboration | 多个角色 agent 分工协作 | Researcher、marketer、editor 一起写 marketing brochure | 行为不确定、协调成本高、debug 更难 |

## Reflection

Reflection 的基本做法：

1. LLM 先生成一个 output，比如代码。
2. 把 output 再交给 LLM 检查 correctness、style、efficiency。
3. LLM 给出 critique。
4. 把 critique 反馈给模型，让它修复或改进 output。

如果能运行代码或单元测试，还可以把 error message/test result 作为外部反馈加入 reflection loop。这样通常比纯粹“自我批评”更可靠。

这不是魔法，不会让系统 100% 正确，但经常能带来性能提升。

## Tool Use

Tool use 的关键是让 LLM 不只生成文本，还能调用函数完成工作。

例子：

- 问“最好的咖啡机是什么？”时，LLM 可以调用 web search 获取新信息。
- 问复利计算时，LLM 可以写代码并执行，得到更可靠的数字结果。
- 在业务系统里，LLM 可以查 database、发 email、看 calendar、处理图片或获取实时数据。

Tool use 能显著扩大模型能力，但也意味着必须设计工具边界、权限、日志、错误处理和 eval。

## Planning

Planning 指 LLM 自己决定完成任务所需的步骤。

课程提到的例子是：用户要求生成一张新图片，并基于另一张图片的 pose，同时还要语音描述结果。系统可能需要自己规划：

1. 用 pose detection model 识别参考图姿势。
2. 用 image generation 生成新图片。
3. 用 image-to-text 描述图片。
4. 用 text-to-speech 生成语音。

Planning 的关键差异是：步骤不是开发者提前 hard-code，而是模型运行时决定。它更灵活，但也更难控制、调试和评估。

## Multi-Agent Collaboration

Multi-agent workflow 是让多个 agent 扮演不同角色并协作。

例子：

- 软件开发：CEO、programmer、tester、designer 等角色协作。
- Marketing brochure：researcher 做资料调研，marketer 写文案，editor 润色。
- Reflection 也可以变成两个 agent：writer agent 生成，critic agent 批评和测试。

这里的 agent 通常只是“同一个或不同 LLM + 不同 role prompt”。它们像一个虚拟团队一样分工，但也会带来更多协调和可控性问题。

## Design Takeaways

- 这四个 design patterns 可以和前面学到的 autonomy spectrum 串起来看：reflection/tool use 往往比较容易落地，planning/multi-agent 通常 autonomy 更高、更难控。
- Pattern 不是越多越好。每加入一个 pattern，都应该问：它解决了哪个具体 failure mode？
- Reflection 适合有可检查 output 的任务，比如代码、文章、答案草稿。
- Tool use 适合模型缺少外部信息、需要计算、需要操作业务系统的任务。
- Planning 适合步骤不固定的开放任务，但需要更强 trace 和 eval。
- Multi-agent 适合需要多种角色视角的复杂任务，但不应该用来掩盖任务拆解不清。

## Failure Modes

- Reflection 没有外部信号，只是模型自说自话，可能越改越偏。
- Tool use 没有权限和审批边界，可能造成错误外部动作。
- Planning 让模型自由决定步骤，但没有 trace、stopping condition 或 component eval。
- Multi-agent 系统看起来丰富，但角色分工不清，输出互相覆盖或循环争论。
- 为了“更 agentic”堆叠模式，导致系统比任务本身还复杂。

## Implementation Hooks

- Reflection: 给模型明确 critique rubric，并尽量加入 test result、error message、user feedback 等外部信号。
- Tool use: 为每个 tool 定义 schema、权限、错误处理、日志和 human approval 规则。
- Planning: 要记录 plan、每一步 action、工具调用结果和 re-planning 原因。
- Multi-agent: 给每个 agent 明确 role、input/output、handoff boundary 和停止条件。
- Evals: 对每个 pattern 都设计对应 eval，例如 reflection 是否提升质量、tool call 是否正确、plan 是否合理、agent collaboration 是否减少错误。

## My Questions

- 我的项目里哪些问题适合先用 reflection，而不是上来做 planning 或 multi-agent？
- Tool use 的权限边界和 human approval 应该如何设计？
- Planning 的 plan quality 应该如何 eval？
- Multi-agent workflow 什么时候真的有价值，什么时候只是增加复杂度？

## Related

- [Degrees of Autonomy](03-degrees-of-autonomy.md)
- [Task Decomposition](06-task-decomposition-identifying-the-steps-in-a-workflow.md)
- [Evaluating Agentic AI](07-evaluating-agentic-ai-evals.md)
- [Agentic design patterns](../../concepts/agentic-design-patterns.md)
- [Open questions](../../review/open-questions.md)

## Next Actions

- [x] 抽取 agentic design patterns concept card。
- [ ] Module 2 学 reflection 后，回头扩展 reflection 部分。
- [ ] 学到 tool use、planning、multi-agent 模块后，分别补充更具体的实现细节。

