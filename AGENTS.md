# Agent Instructions

This file gives project-specific instructions to Codex or any coding agent working in this repository.

## Scope

These instructions apply to the whole repository.

## Project Goal

This is a learning repository for the DeepLearning.AI Agentic AI course. The goal is to build a personal, durable learning system:

- course notes in `notes/`
- reusable concept cards in `concepts/`
- experiments in `labs/`
- review material in `review/`
- external references in `references/`

Do not turn this into a transcript dump.

## First Files To Read

Before making structural changes, read:

1. `MEMORY.md`
2. `README.md`
3. `SOP.md`
4. current module README under `notes/`
5. relevant file under `references/maps/` if external code is involved

## Note-Writing Rules

- Write primarily in Chinese, keeping important English technical terms when they are useful.
- Summarize course transcripts; do not preserve full transcripts verbatim.
- Keep notes practical: design takeaways, failure modes, implementation hooks, open questions.
- When a concept appears across lessons, create or update a card in `concepts/`.
- When a question remains unresolved, add it to `review/open-questions.md`.
- When a term becomes recurring, add it to `review/glossary.md`.
- Update the module checklist and note links in the module README.

## External Reference Rules

- `references/upstream/` contains external repositories and should be treated as read-only.
- `references/upstream/agentic_ai_andrew/` is a git submodule.
- Do not modify upstream code for personal experiments.
- When using upstream code as inspiration, create a lab under `labs/` and document the source path.
- Maintain navigation files under `references/maps/`.

## Lab Rules

- Labs should be personal reproductions or experiments, not blind copies.
- Each lab should explain:
  - goal
  - source inspiration
  - setup
  - workflow
  - observations
  - eval or success criteria
  - next iteration
- Prefer minimal reproductions before full-stack rewrites.

## Git/Submodule Notes

For a fresh clone:

```bash
git submodule update --init --recursive
```

To update the external reference repo:

```bash
git submodule update --remote references/upstream/agentic_ai_andrew
```

Before committing, check:

```bash
git status --short
git submodule status
```

## What To Avoid

- Do not revert or delete user notes.
- Do not overwrite unfinished lesson notes.
- Do not directly edit `references/upstream/` unless explicitly requested.
- Do not add full copyrighted course transcripts.
- Do not add heavyweight generated files unless the user explicitly wants them.
- Do not create labs that require paid API keys without documenting the requirement clearly.

## Current Reference Map

Use:

```text
references/maps/agentic_ai_andrew-map.md
```

It maps the external `nhatnam2609/agentic_ai_andrew` repository to this repo's notes, concepts, and future labs.

