# long-task-guidance

[中文 README](README.zh.md) | [SKILL Specification](SKILL.md)

An anti-drift execution specification for agent long-running tasks (full spec: [SKILL.md](SKILL.md)): externalize goals, specs, progress, and decisions into files to counteract context compression and goal drift, so the task can be losslessly resumed at any interruption point.

## Key Features

- **Tree-structured documents**: overall directory → feature modules → submodules, refined level by level; one directory = one work unit, and the doc tree grows with the task
- **Files as memory**: goals, specs, progress, and decisions all live on disk; lossless resumption after context compression or interrupted sessions
- **Spec-first + TDD**: no implementation until the spec is settled; acceptance checklist items are numbered and individually decidable
- **Tabular progress management**: Plan-and-Progress is mandatory task table + subtask table, with four-state status (Not Started / In Progress / Done / Blocked) + plan zero-out—no dangling tasks
- **Incremental change ledger**: every requirement/spec change is mandatorily logged; history is replayable

## Best Usage

**You do not need to keep this skill permanently mounted in your coding agent.** Recommended usage:

1. Before starting a task, copy `SKILL.md` into the target project;
2. Have the agent read it and follow the workflow it defines;
3. The agent will generate and continuously maintain the full set of task documents inside the project.

The skill is a one-shot injected execution spec—it does not consume resident context; all task state lives in the documents, decoupled from the skill itself.

## When to Use

- **Long tasks**: complex tasks advanced across multiple sessions and stages. Files are memory: after context compression or an interrupted conversation, `STATE.md` plus the spec files are enough to resume losslessly.
- **Short tasks (human–agent collaborative docs)**: even small tasks can use this spec to make the agent write docs to disk—people review files to understand and steer progress instead of scrolling chat logs; the documents are the collaboration interface.
- **Vibecoding**: when driving code with natural language, "spec first + numbered acceptance checklist" gives the vibe an objective anchor; the change ledger keeps requirement changes replayable and prevents gradual drift.

## The Workflow

Every work unit (one directory = one work unit) proceeds in this order:

| # | Step | Key point |
|---|------|-----------|
| 1 | Research first | Survey current practice before starting; avoid outdated intuition |
| 2 | Spec first | No implementation until the spec is settled |
| 3 | Tree organization | Overall dir → feature modules → submodules, refined per level |
| 4 | TDD implementation | Tests before implementation; done only when tests pass |
| 5 | Serial completion | Fully finish the current unit before the next (incl. plan zero-out) |
| 6 | Incremental logging | Requirement/spec changes must be logged; no silent overwrites |
| 7 | Architectural discipline | No patches on patches; refactor when patchwork piles up |
| 8 | Delivery | Each unit produces a Feature-Doc + Principle-Doc |

Supporting mechanisms:

- **Anti-compression reread protocol**: before a new unit, after finishing a subtask, and after context compression, the status and spec files must be reread before proceeding.
- **Commits and checkpoints**: one commit per finished work unit; commit messages reference doc paths.

## Agent Deliverables

```text
<task-doc root>/
├── STATE.md                         # top-level status entry (unique)
├── ENVIRONMENT.md                   # user-written environment notes (never in git)
└── 00-Overall-<project>/
    ├── Research.md / Requirements.md / Architecture.md
    ├── Plan-and-Progress.md / Change-Log.md
    ├── <feature-module>/
    │   ├── Requirements.md / Spec.md (numbered acceptance checklist)
    │   ├── Plan-and-Progress.md / Change-Log.md
    │   └── <submodule>/             # same structure, finer-grained
    └── Deliverables/<feature-module>/
        ├── Feature-Doc.md           # what it does, how to use
        └── Principle-Doc.md         # principles, end-to-end chain
```

| Level | Deliverables |
|-------|--------------|
| Top level | `STATE.md`, `ENVIRONMENT.md` |
| Overall directory | Research, Requirements, Architecture, Plan-and-Progress, Change-Log |
| Feature module / submodule | Requirements, Spec, Plan-and-Progress, Change-Log |
| Deliverables level | Feature-Doc, Principle-Doc |

**Plan-and-Progress** is mandatory table format (task table + subtask table) with a Status column (Not Started / In Progress / Done / Blocked), mapped to spec checklists via item numbers; cross-module and integration-phase tasks are registered only in the overall plan.
