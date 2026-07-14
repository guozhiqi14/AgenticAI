# Reference Map: nhatnam2609/agentic_ai_andrew

Source: `references/upstream/agentic_ai_andrew/`

这个文件是我们自己的导航索引。`upstream/agentic_ai_andrew/` 保持原样；这里负责记录它和你的学习笔记、concept cards、labs 之间的关系。

## How To Use This Repo

1. 先看你自己的课程笔记：`notes/` 和 `concepts/`。
2. 再来这里找对应的 upstream lecture notes、notebooks 或代码。
3. 如果要运行或修改代码，不要直接改 upstream；在 `labs/` 下新建你自己的实验。
4. 每次做 lab 时，在 lab note 里写清楚参考了 upstream 哪个文件、你改了什么、学到了什么。

## Upstream Structure

| Upstream Path | What It Contains | How We Use It |
| --- | --- | --- |
| `Lecture_notes/` | M1-M5 PDF lecture notes | 对照课程概念，补充复习材料 |
| `Module1/agentic-ai-public/` | FastAPI + Postgres reflective research agent | 作为 research agent / planning workflow 的代码参考 |
| `Module2/` | Reflection design pattern notebooks and labs | 学 Module 2 时对照 reflection 笔记和 lab |
| `Module3/` | Tool use design pattern, function calling, research tools | 学 tool use 时对照工具定义和调用方式 |
| `Module4/` | Evals and practical tips | 学 evals 时回补 `concepts/agentic-evals.md` |
| `Module5/` | Multi-agent collaboration and final project | 学 multi-agent 时做架构参考 |

## Map To Our Notes

| Our Note | Upstream Reference | Use |
| --- | --- | --- |
| `notes/module-01-introduction-to-agentic-workflows/03-degrees-of-autonomy.md` | `Module1/agentic-ai-public/src/planning_agent.py` | 看更高 autonomy 下 planner/executor 如何分工 |
| `notes/module-01-introduction-to-agentic-workflows/04-benefits-of-agentic-ai.md` | `Module1/agentic-ai-public/README.md` | 对照 performance、parallelism、modularity 如何体现在完整 app |
| `notes/module-01-introduction-to-agentic-workflows/05-agentic-ai-applications.md` | `Module1/agentic-ai-public/` | 作为一个完整 agentic application 例子 |
| `notes/module-01-introduction-to-agentic-workflows/06-task-decomposition-identifying-the-steps-in-a-workflow.md` | `Module1/agentic-ai-public/src/agents.py` and `src/planning_agent.py` | 对照 planner、researcher、writer、editor 的步骤拆解 |
| `notes/module-01-introduction-to-agentic-workflows/07-evaluating-agentic-ai-evals.md` | `Module4/` | 等学到 Module 4 后补充 evals 实战细节 |
| `notes/module-01-introduction-to-agentic-workflows/08-agentic-design-patterns.md` | `Module2/`, `Module3/`, `Module5/` | 分别对应 reflection、tool use、multi-agent |
| `notes/module-02-reflection-design-pattern/01-reflection-to-improve-outputs-of-a-task.md` | `Module2/M2_README.md` and `Module2/` labs | 对照 reflection 的 draft -> critique -> revise 实例 |
| `notes/module-02-reflection-design-pattern/02-why-not-just-direct-generation.md` | `Module2/M2_README.md` and reflection labs | 用 upstream labs 做 direct generation vs reflection 对照 |
| `notes/module-03-tool-use/01-what-are-tools.md` | `Module3/M3_README.md` and `Module3/` labs | 对照 function calling、tool schema 和多工具工作流 |
| `notes/module-03-tool-use/02-creating-a-tool.md` | `Module3/M3_README.md` and `Module3/` labs | 对照 tool registry、tool call request 和 tool result 回填 |
| `notes/module-03-tool-use/03-tool-syntax.md` | `Module3/M3_README.md` and `Module3/` labs | 对照 tool schema、function docstring 和 tool-call loop 实现 |
| `notes/module-03-tool-use/06-code-execution.md` | `Module3/M3_README.md` and `Module3/` labs | 对照 code execution tool、sandbox、error feedback 和 retry loop |
| `notes/module-03-tool-use/07-mcp.md` | `Module3/M3_README.md` and `Module3/` labs | 对照 MCP client/server、resources、tool ecosystem 和标准化集成 |
| `notes/module-04-evals-and-optimization/01-evaluations-evals.md` | `Module4/M4_README.md` | 课程页为主；upstream Module 4 当前只作后续 eval/lab 方向提示 |
| `notes/module-04-evals-and-optimization/02-error-analysis-and-prioritizing-next-steps.md` | `Module4/M4_README.md` | 课程页为主；用作后续 component-level eval lab 的 error-analysis checklist |
| `notes/module-04-evals-and-optimization/03-more-error-analysis-examples.md` | `Module4/M4_README.md` | 课程页为主；补充 invoice/customer-email 的 component attribution 案例 |
| `notes/module-04-evals-and-optimization/04-component-level-evaluations.md` | `Module4/M4_README.md` | 课程页为主；为后续 research workflow component-level eval lab 铺垫 |
| `notes/module-04-evals-and-optimization/06-how-to-address-problems-you-identify.md` | `Lecture_notes/agentic_workflows_M4_learner.pdf` | 课程页 transcript 为主；lecture PDF 可辅助复习 non-LLM/LLM component 修法菜单 |
| `notes/module-04-evals-and-optimization/07-latency-cost-optimization.md` | `Lecture_notes/agentic_workflows_M4_learner.pdf` | 课程页 transcript 为主；lecture PDF 可辅助复习 per-step timing/cost breakdown |
| `notes/module-04-evals-and-optimization/08-development-process-summary.md` | `Lecture_notes/agentic_workflows_M4_learner.pdf` | 课程页 transcript 为主；lecture PDF 可辅助复习 build/analyze development loop |
| `notes/zhiqi_notes.md` | `Module3/ungraded_labs/M3_UGL_1/`, `Module3/ungraded_labs/M3_UGL_2/` | 记录手动 tool-call loop、message history、REST route 和 backend mental model |

## Code Reading Path

优先从 Module 1 的 research agent 开始读，因为它和你现在的 Module 1 笔记最贴近。

1. `Module1/agentic-ai-public/README.md`
   - 先理解这个 app 的目标、API 和运行方式。
2. `Module1/agentic-ai-public/main.py`
   - 看 FastAPI 如何启动 workflow、记录 task status、暴露 progress API。
3. `Module1/agentic-ai-public/src/planning_agent.py`
   - 看 planner 如何把任务拆成步骤，以及 executor 如何推进 workflow。
4. `Module1/agentic-ai-public/src/agents.py`
   - 看 research、writer、editor 这些角色如何被实现。
5. `Module1/agentic-ai-public/src/research_tools.py`
   - 看 Tavily、arXiv、Wikipedia 等工具如何封装。

## Suggested Labs To Create Later

| Lab | Source Inspiration | Goal |
| --- | --- | --- |
| `labs/001-research-agent-minimal/` | `Module1/agentic-ai-public/` | 做一个最小 research agent，不先引入完整 FastAPI/Postgres |
| `labs/002-reflection-loop/` | `Module2/` | 复现 reflection: draft -> critique -> revise |
| `labs/003-tool-use-minimal/` | `Module3/` | 练习 function calling / tool schema |
| `labs/004-agent-evals/` | `Module4/` | 建 objective eval + LLM-as-judge 对照 |
| `labs/005-multi-agent-pipeline/` | `Module5/` | 做一个小型多角色 pipeline |

## Reading Notes

- Upstream README 把 Module 1 描述为 planning/research agent，这和我们 Module 1 的 introduction notes 可以互相补充。
- Upstream 的 Module 2/3/5 对应课程后续 design patterns，暂时不急着读透。
- 当前课程页的 Module 4 是 evals / error analysis / optimization 主线；本地 upstream `Module4/M4_README.md` 内容较稀疏且不完全对齐，先以课程页面和自己的笔记为主，等做 component-level eval lab 时再决定是否借用 upstream。

## Maintenance

更新 submodule:

```bash
git submodule update --remote references/upstream/agentic_ai_andrew
```

初次 clone 这个学习仓库后拉取 submodule:

```bash
git submodule update --init --recursive
```
