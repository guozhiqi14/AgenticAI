# Project Memory

这个文件记录项目的长期上下文。新机器、新会话或新的 agent 接手时，先读这里。

## Project Purpose

把 DeepLearning.AI 的 Agentic AI 课程沉淀成个人工作手册，而不是 transcript 仓库。

核心产物包括：

- Lesson notes: 按课程顺序记录理解。
- Concept cards: 抽象出可复用的概念和设计判断。
- Labs: 复现、改造或验证课程里的 agentic workflow。
- Review material: open questions、glossary、module summaries、自测题。
- References: 外部课程笔记和代码，只作为参考资料。

## Current Learning State

当前主线课程：DeepLearning.AI Agentic AI。

当前已重点整理 Module 1:

- `03-degrees-of-autonomy.md`
- `04-benefits-of-agentic-ai.md`
- `05-agentic-ai-applications.md`
- `06-task-decomposition-identifying-the-steps-in-a-workflow.md`
- `07-evaluating-agentic-ai-evals.md`
- `08-agentic-design-patterns.md`

Module 1 当前核心概念卡：

- `concepts/autonomy-spectrum.md`
- `concepts/agentic-workflow-benefits.md`
- `concepts/agentic-application-fit.md`
- `concepts/task-decomposition.md`
- `concepts/agentic-evals.md`
- `concepts/agentic-design-patterns.md`

Module 2 已开始：

- `notes/module-02-reflection-design-pattern/01-reflection-to-improve-outputs-of-a-task.md`
- `notes/module-02-reflection-design-pattern/02-why-not-just-direct-generation.md`
- `notes/module-02-reflection-design-pattern/05-evaluating-the-impact-of-reflection.md`
- `notes/module-02-reflection-design-pattern/06-using-external-feedback.md`

Module 2 当前核心概念卡：

- `concepts/reflection-pattern.md`
- `concepts/agentic-evals.md`
- `concepts/external-feedback.md`

Module 3 已开始：

- `notes/module-03-tool-use/01-what-are-tools.md`
- `notes/module-03-tool-use/02-creating-a-tool.md`
- `notes/module-03-tool-use/03-tool-syntax.md`
- `notes/module-03-tool-use/06-code-execution.md`
- `notes/module-03-tool-use/07-mcp.md`

Module 3 当前核心概念卡：

- `concepts/tool-use.md`
- `concepts/tool-schema.md`
- `concepts/code-execution-tool.md`
- `concepts/model-context-protocol.md`

Module 4 已开始：

- `notes/module-04-evals-and-optimization/01-evaluations-evals.md`
- `notes/module-04-evals-and-optimization/02-error-analysis-and-prioritizing-next-steps.md`
- `notes/module-04-evals-and-optimization/03-more-error-analysis-examples.md`
- `notes/module-04-evals-and-optimization/04-component-level-evaluations.md`
- `notes/module-04-evals-and-optimization/06-how-to-address-problems-you-identify.md`
- `notes/module-04-evals-and-optimization/07-latency-cost-optimization.md`
- `notes/module-04-evals-and-optimization/08-development-process-summary.md`

Module 4 当前核心概念卡：

- `concepts/agentic-evals.md`
- `concepts/error-analysis.md`
- `concepts/component-level-eval.md`
- `concepts/latency-cost-optimization.md`
- `concepts/agentic-development-loop.md`

Module 5 已开始：

- `notes/module-05-patterns-for-highly-autonomous-agents/01-planning-workflows.md`
- `notes/module-05-patterns-for-highly-autonomous-agents/02-creating-and-executing-llm-plans.md`
- `notes/module-05-patterns-for-highly-autonomous-agents/03-planning-with-code-execution.md`

Module 5 当前核心概念卡：

- `concepts/planning-pattern.md`
- `concepts/agentic-design-patterns.md`
- `concepts/autonomy-spectrum.md`
- `concepts/task-decomposition.md`
- `concepts/tool-use.md`
- `concepts/code-execution-tool.md`

横向个人笔记：

- `notes/zhiqi_notes.md`

## Current Repository Design

```text
notes/       course-order notes
concepts/    reusable concept cards
labs/        personal experiments and runnable reproductions
review/      glossary, summaries, open questions, review prompts
templates/   reusable note templates
references/  external read-only references and maps
```

External upstream repo:

```text
references/upstream/agentic_ai_andrew/
```

It is a git submodule pointing to:

```text
https://github.com/nhatnam2609/agentic_ai_andrew.git
```

Use the map:

```text
references/maps/agentic_ai_andrew-map.md
```

## Durable Decisions

- Do not store full course transcripts in this repo.
- Summarize transcript content into original notes and design takeaways.
- Keep the user's own notes as the source of truth.
- Treat `references/upstream/` as read-only.
- Put personal experiments in `labs/`, not inside upstream reference repos.
- Use `concepts/` for cross-lesson ideas.
- Use `review/open-questions.md` for unresolved questions.
- Use `review/glossary.md` for terms that recur across lessons.
- Update module README checklists as lessons are completed.
- Treat planning notes as higher-autonomy workflow material: connect them back to tool use, evals, traces, and guardrails.
- For executable plans, prefer structured JSON/XML plan schemas over transcript-like plain text; note parser/validator/executor boundaries.
- For code-as-plan lessons, connect back to pandas/data-analysis use cases but keep sandbox, permissions, timeout, trace, and eval boundaries explicit.

## Default Lesson Note Shape

Each substantial lesson note should usually include:

- `TL;DR`
- `Core Ideas`
- concrete examples from the lesson
- `Design Takeaways`
- `Failure Modes`
- `Implementation Hooks`
- `My Questions`
- `Related`
- `Next Actions`

This is a default, not a rigid law. Keep notes useful.

## Working With Course Pages

When Chrome browser access works:

1. Identify the current course URL and lesson title.
2. Open/show transcript if available.
3. Read enough visible/transcript content to summarize.
4. Do not save full transcript.
5. Create or update the matching lesson note.
6. Update concepts/review/module checklist as needed.

When browser access fails:

- Ask the user to paste transcript, screenshots, or rough bullet notes.
- Continue with whatever material is available.

## Working With External References

External references are used in this order:

1. Read `references/maps/...` to understand relevance.
2. Read upstream README or selected code file.
3. Link insights back to `notes/`, `concepts/`, or `labs/`.
4. If experimentation is needed, create a new folder under `labs/`.

Never directly mutate upstream reference code unless the user explicitly asks to patch the external repo itself.

## New Machine Setup

After cloning this repo:

```bash
git submodule update --init --recursive
```

Then read:

1. `README.md`
2. `MEMORY.md`
3. `AGENTS.md`
4. `SOP.md`
5. `references/maps/agentic_ai_andrew-map.md`

## Next Likely Work

- Continue with the remaining Module 1 supporting items if the user wants them:
  - local environment setup reading
  - module quiz review
  - research agent code example
- Continue Module 2: Reflection Design Pattern.
- Continue Module 3 quiz / graded lab if the user wants to close pending course items.
- Continue Module 4 pending items only when needed:
  - `Ungraded Lab: Adding a component-level eval to the research workflow`
  - `Module 4 quiz`
- Continue Module 5: Patterns for Highly Autonomous Agents.
- Next likely Module 5 video: `Multi-agentic workflows`.
- Create first lab from upstream Module 1 research agent:
  - proposed path: `labs/001-research-agent-minimal/`
