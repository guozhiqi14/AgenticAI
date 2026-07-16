# Concept: Multi-Agent Communication Patterns

## Definition

Multi-agent communication pattern 是 multi-agent workflow 里 agents 之间交换信息、委派任务、回传结果和决定完成状态的结构。

它回答的不是“有哪些 agents”，而是：

- 谁可以和谁通信？
- 信息按什么顺序传？
- 谁负责整合？
- 什么时候停止？
- 出错时如何定位责任？

## Why It Matters

同样是 researcher、designer、writer 三个 agents，不同通信模式会产生完全不同的系统行为。

Communication pattern 直接影响：

- controllability
- latency/cost
- trace readability
- debugging difficulty
- final output quality
- 是否适合高风险业务场景

## Common Patterns

### Linear Handoff

```text
agent A -> agent B -> agent C -> final output
```

适合固定顺序任务。优点是简单、可控、好 debug；缺点是上游错误会一路传下去，不适合反复协商。

### Hierarchical / Manager Coordination

```text
manager
  -> specialist A
  -> specialist B
  -> specialist C
```

Manager agent 负责委派、收集结果和最终整合。它比 linear 更灵活，但 manager 的 delegation 和 integration quality 本身需要 eval。

### Deeper Hierarchy

```text
manager
  -> specialist
      -> sub-agent 1
      -> sub-agent 2
```

当某个 specialist 内部也很复杂时，可以继续拆 sub-agents。这个模式更强，但 trace、latency、cost 和出错定位都会变难。

### All-to-All

```text
agent A <-> agent B
agent A <-> agent C
agent B <-> agent C
```

任意 agent 可以和任意 agent 通信。它适合探索和创意任务，但高度不可预测，不适合需要稳定控制和清晰责任链的场景。

## Selection Heuristic

优先级可以这样想：

1. 任务顺序固定：用 linear。
2. 需要动态委派和最终整合：用 manager hierarchy。
3. 某个 specialist 内部复杂且可复用：考虑 deeper hierarchy。
4. 需要开放探索且能容忍混乱：才考虑 all-to-all。

不要因为系统是 multi-agent，就默认做成 all-to-all。多数实际应用更需要可控通信，而不是最大自由度。

## Design Checklist

- Agents: 有哪些角色？
- Allowed routes: 谁能给谁发消息？
- Handoff schema: 每次传递包含哪些字段？
- Integration owner: 谁负责最终整合？
- Stop condition: 什么时候结束？
- Trace: 如何记录每条 message？
- Eval: 如何评价 delegation、handoff、specialist output 和 final output？
- Limits: 最大轮数、最大层级、最大成本是多少？

## Failure Modes

- Communication path 太自由，agents 聊天但不收敛。
- Communication path 太死，无法根据中间结果调整。
- Manager 变成 bottleneck。
- Handoff 缺少结构，接收方误解上游输出。
- Deeper hierarchy 让责任链过长，debug 成本飙升。
- All-to-all 没有 stopping condition，导致循环或随机结束。

## Data Analyst Translation

数据分析 workflow 可以先用 linear：

```text
metric definition -> SQL -> statistics -> visualization -> insight writing
```

当任务需要动态判断时，再引入 manager：

```text
analysis manager
  -> metric analyst
  -> SQL analyst
  -> statistician
  -> visualization designer
  -> insight writer
```

All-to-all 更适合 hypothesis brainstorming 或报告 critique，不适合直接自动生成可交付业务结论。

## Related Notes

- [Communication patterns for multi-agent systems](../notes/module-05-patterns-for-highly-autonomous-agents/07-communication-patterns-for-multi-agent-systems.md)
- [Multi-agentic workflows](../notes/module-05-patterns-for-highly-autonomous-agents/05-multi-agentic-workflows.md)
- [Multi-agent collaboration](multi-agent-collaboration.md)
- [Planning pattern](planning-pattern.md)
- [Agentic evals](agentic-evals.md)
