# Module 1 Summary

Status: in progress

## Working Thesis

Agentic workflow 最好理解成一种“让 LLM 参与 workflow 决策”的系统。关键设计动作是决定：给多少 autonomy、在哪里使用 tools、如何评估 intermediate actions 和 final outputs。

## Lessons Covered

- [x] Degrees of autonomy
- [x] Benefits of Agentic AI
- [x] Agentic AI Applications
- [x] Task decomposition
- [x] Evaluating Agentic AI
- [x] Agentic design patterns

## Ideas To Revisit

- Autonomy as a spectrum。
- Tool use as a control boundary。
- task decomposition 和 eval design 的关系。
- Agentic workflow 的收益：performance、parallelism、modularity。
- 应用选择判断：SOP、文本输入、稳定 API、human review、可评估步骤。
- Task decomposition 的核心问题：每一步能否由 LLM、tool、API、code 或 human review 实现？
- Evals 的核心流程：build、inspect outputs/traces、find failures、write evals、iterate。
- Evals 的两条轴：objective vs subjective，以及 end-to-end vs component-level。
- 四个核心 design patterns：reflection、tool use、planning、multi-agent collaboration。

## Self-Test Questions

- less autonomous workflow 和 highly autonomous workflow 的区别是什么？
- 为什么生产环境里可能更适合低自主性的 workflow？
- 当 LLM 可以选择 tools 或创造步骤时，会出现哪些新风险？
- agentic workflow 相比 single-prompt baseline 可能有哪些收益？
- 为什么 modularity 对 agentic system 的迭代很重要？
- 为什么已有 SOP 的任务更适合作为 agentic workflow 的起点？
- computer use agent 为什么比 API-based agent 更难稳定？
- task decomposition 时，为什么要先思考“人类会怎么做”？
- 当某个 workflow 输出不好时，为什么可以只拆解表现差的那一步？
- 为什么 agentic workflow 需要 traces？
- objective eval 和 LLM-as-judge 分别适合什么场景？
- end-to-end eval 和 component-level eval 的区别是什么？
- reflection、tool use、planning、multi-agent collaboration 分别解决什么问题？
- 为什么 planning 和 multi-agent 通常比 reflection/tool use 更难控制？
