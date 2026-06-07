# Agentic AI Learning Notes

这个仓库用来把 DeepLearning.AI 的 Agentic AI 课程沉淀成你自己的工作手册。

目标不是保存逐字稿，而是把每节课转成：概念、设计判断、取舍、实验、问题和复习材料。最后你应该得到一套能拿来设计 agent workflow 的笔记，而不只是一堆视频摘要。

## 快速开始

如果你是在一台新机器上 clone 这个仓库：

```bash
git clone <this-repo-url>
cd AgenticAI
git submodule update --init --recursive
```

然后按这个顺序读：

1. [MEMORY.md](MEMORY.md): 项目当前状态、设计决策、学习进度。
2. [SOP.md](SOP.md): 你和 Codex 如何协作看课、记笔记、做 lab。
3. [AGENTS.md](AGENTS.md): 给 Codex/其他 coding agent 的项目规则。
4. [Module 1 notes](notes/module-01-introduction-to-agentic-workflows/README.md): 当前课程主线进度。
5. [External reference map](references/maps/agentic_ai_andrew-map.md): 外部 GitHub 仓库和我们笔记的对应关系。

## 目录结构

- `notes/`: 按课程顺序记录每节课。
- `concepts/`: 按主题沉淀可复用概念卡片。
- `labs/`: 记录实验、代码、prompt、eval 和观察。
- `templates/`: 每类笔记的模板。
- `review/`: 术语表、开放问题、模块总结和自测题。
- `references/`: 外部课程资料和代码仓库，作为 read-only reference 使用。
- `SOP.md`: 你之后如何和 Codex 配合看课。
- `MEMORY.md`: 项目状态、上下文和长期约定。
- `AGENTS.md`: agent 接手本项目时必须遵守的工作规则。

## 学习闭环

1. 每次只处理一节课，笔记放进 `notes/`。
2. 把可复用的想法抽到 `concepts/`。
3. 如果课程有 lab 或代码例子，就在 `labs/` 里记录实验。
4. 没想明白的问题先放进 `review/open-questions.md`。
5. 每个 module 结束后，在 `review/` 里做一次总结和自测。

## 协作原则

- 你的笔记永远是主线，外部资料只是 reference。
- 课程 transcript 只用于总结和理解，不把完整逐字稿原样保存进仓库。
- `references/upstream/` 下的外部仓库保持 read-only 心态；实验代码写到 `labs/`。
- 每篇 lesson note 尽量包含：TL;DR、Core Ideas、Examples、Design Takeaways、Failure Modes、Implementation Hooks、My Questions。
- 每个重要概念尽量抽到 `concepts/`，方便跨章节复用。
- 每个 module 结束后更新 `review/module-XX-summary.md` 或对应 summary。

## 当前进度

Course: Agentic AI  
Module 1: Introduction to Agentic Workflows

Module 1 已整理：

- Degrees of autonomy
- Benefits of agentic AI
- Agentic AI applications
- Task decomposition
- Evaluating agentic AI
- Agentic design patterns

建议从这里开始：

- [MEMORY.md](MEMORY.md)
- [AGENTS.md](AGENTS.md)
- [SOP.md](SOP.md)
- [Module 1 notes](notes/module-01-introduction-to-agentic-workflows/README.md)
- [Autonomy spectrum concept card](concepts/autonomy-spectrum.md)
- [External reference map](references/maps/agentic_ai_andrew-map.md)
