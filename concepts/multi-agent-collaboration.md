# Concept: Multi-Agent Collaboration

## Definition

Multi-agent collaboration 是把复杂任务拆给多个 agents 协作完成的 workflow pattern。每个 agent 通常有自己的 role prompt、工具权限、输入输出格式和责任边界。

它不是简单地“多调用几次 LLM”，而是把复杂任务做 role decomposition：谁负责研究、谁负责计算、谁负责写作、谁负责审稿、谁负责协调。

## Why It Matters

复杂任务常常天然不是一个角色能做完的。人类团队会分工，软件系统也可以按职责拆分。

Multi-agent collaboration 的价值在于：

- 降低单个 prompt 的复杂度。
- 让每个 agent 聚焦一个能力。
- 让不同 agent 拥有不同 tools 和权限。
- 支持 per-agent debug 和 eval。
- 让成熟 agent 有机会在不同 workflow 里复用。

## Common Shapes

```text
linear:
  user -> agent A -> agent B -> agent C -> output

manager:
  user -> manager agent -> specialist agents -> manager review -> output
```

线性模式更简单，适合固定顺序任务。Manager 模式更灵活，适合需要动态委派、整合和 review 的任务。

更完整地看，multi-agent system 的通信模式还包括 deeper hierarchy 和 all-to-all。通信模式决定上下文如何流动、谁负责整合、什么时候停止，也决定系统是否容易 debug 和 eval。

## Agents As Tools

Manager-agent workflow 可以看成 planning pattern 的升级版：

```text
tool planning:
  planner chooses tools

agent planning:
  manager chooses specialist agents
```

一个 specialist agent 像一个更高级的 tool：它不只是函数，而是带 role prompt、tools、memory/trace 和局部 workflow 的小系统。

## Role Design

一个 agent role 至少要定义：

- Responsibility: 它负责什么。
- Inputs: 它需要什么上下文。
- Outputs: 它交付什么格式。
- Tools: 它允许调用哪些工具。
- Boundaries: 它不能做什么。
- Eval: 怎样判断它做得好不好。

Role boundary 越清楚，multi-agent workflow 越容易调试。

## When To Use

- 任务天然对应多个专业角色。
- 不同子任务需要不同工具或权限。
- 某些能力值得单独优化或复用。
- 单个 agent prompt 已经太长、职责太混乱。
- 需要独立 review/editor/critic 角色。

## When To Avoid

- 任务可以用单个 deterministic workflow 稳定完成。
- 多 agent 只是为了显得更 agentic。
- 子任务边界不清楚，agent 会互相重复或推责。
- 没有 trace/eval，无法判断哪个 agent 出错。
- latency/cost 对业务非常敏感。

## Failure Modes

- Role overlap: 多个 agent 做重复工作。
- Coordination overhead: 沟通比工作本身更复杂。
- Context bloat: 每个 agent 都传大量上下文，成本和噪音上升。
- Blame ambiguity: final output 错了，但不知道是哪个 agent 造成。
- Manager error: manager 委派错误或漏掉关键 specialist。
- Tool permission drift: 每个 agent 都拿到过多工具，风险扩大。
- Communication chaos: all-to-all 或过深 hierarchy 让 agents 互相通信但难以收敛。

## Practical Rule

先问：

> 这个复杂任务是不是天然需要多个不同角色？

如果只是步骤多，planning 或 fixed workflow 可能就够了。只有当角色边界、工具权限、评估标准都能拆清楚时，multi-agent 才值得引入。

## Related Notes

- [Multi-agentic workflows](../notes/module-05-patterns-for-highly-autonomous-agents/05-multi-agentic-workflows.md)
- [Communication patterns for multi-agent systems](../notes/module-05-patterns-for-highly-autonomous-agents/07-communication-patterns-for-multi-agent-systems.md)
- [Multi-agent communication patterns](multi-agent-communication-patterns.md)
- [Agentic design patterns](agentic-design-patterns.md)
- [Planning pattern](planning-pattern.md)
- [Task decomposition](task-decomposition.md)
- [Tool use](tool-use.md)
- [Agentic evals](agentic-evals.md)
