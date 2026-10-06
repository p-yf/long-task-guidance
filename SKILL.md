---
name: long-task-execution
description: Execution and anti-goal-drift specification for long-running, multi-stage tasks. Externalizes goals, specs, progress, and decisions into files to counteract context compression and length limits; mandates research-first, spec-first, tree-structured docs, TDD, serial acceptance, incremental logging, and a reread protocol, and defines the file layout and per-file content rules for every level of the directory tree. Applies to any complex task an agent must advance across multiple sessions.
---

# Long-Task Execution and Anti-Drift Specification

## Purpose and Scope

Constrains how an agent works on long-running, multi-stage tasks, with two goals:

1. **No goal drift**: task intent and acceptance criteria always live in writing and never decay across conversations.
2. **No lost state**: work content and work paths survive context compression and length limits.

Core mechanism: **externalize task state into files.** Conversation history gets compressed or truncated; files don't. Files therefore serve as replayable persistent memory.

## Core Principles

1. **Files as memory**: all goals, specs, progress, and key decisions must be written to disk; keeping them only in conversation context is forbidden.
2. **Externalized goals**: the task's final goal and every subtask's acceptance criteria must be written into files; when resuming work, files are authoritative—not residual memory.
3. **Single status entry**: always maintain one top-level status file that records exactly four items: current phase, current task, next action, known blockers.
4. **Resumable at any breakpoint**: each finished work unit forms a progress anchor (doc update + code commit), so the task can be losslessly resumed after any interruption.

## Preconditions for Starting a Task

Before formally starting, these conditions must be met; otherwise the workflow must not begin:

1. **Name the task-doc root directory**: the user provides the name of the task-doc root directory; all documents are written under it in a tree structure.
2. **User describes the environment**: the user must fill in `ENVIRONMENT.md` in the root directory describing the development environment; the agent refers to it throughout.

## Execution Workflow

For every work unit, execute in this order:

1. **Research first**: before starting work, survey current theory and practice in the relevant domain to avoid designing from outdated intuition. Research only before formally starting, or during communication with the user.
2. **Spec first**: before writing any code, write the unit's spec document; no implementation until the spec is settled.
3. **Tree-structured organization**: documents are organized as a tree—an overall directory (named `Overall-<project/module>`) containing feature modules, each subdivided into submodules; every level gets its own directory and documents.
4. **TDD implementation**: code test-first—write tests before implementation; a step counts as done only when its tests pass.
5. **Serial completion**: fully finish the current unit before starting the next. Completion is judged by objective conditions: deliverable docs are ready, features are defect-free, logic is self-consistent, and the plan is zeroed out (no lingering In-Progress tasks in this unit's plan; see the plan zero-out rule under Plan-and-Progress.md).
6. **Incremental logging**: any change to requirements or specs must be logged incrementally; silently overwriting history is forbidden.
7. **Architectural discipline**: keep the architecture sound and maintainable; no patches on patches. When patchwork piles up, go back and refactor instead of stacking more.
8. **Delivery**: after each unit, produce two documents—Feature-Doc (what it does, how to use it) and Principle-Doc (what principles it relies on, how the whole chain works).

## Layered Directory and File Layout

The doc tree is the persistent memory of task state and must be a tree: overall directory (`Overall-<project>`, holding overall docs + all feature modules + deliverables) → feature modules → submodules, refined level by level.

**Definition: one work unit = one directory at a given level.** The overall directory, each feature-module directory, and each submodule directory are each independent work units; the execution workflow and serial completion advance per directory—one finished directory = one finished work unit.

```text
<task-doc root>/
├── STATE.md                         # top-level status entry (required, unique)
├── ENVIRONMENT.md                   # user-written environment notes (never commit to git)
└── 00-Overall-<project>/            # overall dir: overall docs + feature modules + deliverables
    ├── Research.md
    ├── Requirements.md
    ├── Architecture.md
    ├── Plan-and-Progress.md
    ├── Change-Log.md
    ├── <feature-module>/            # feature modules nested under the overall dir
    │   ├── Requirements.md
    │   ├── Spec.md
    │   ├── Plan-and-Progress.md
    │   ├── Change-Log.md
    │   └── <submodule>/             # finer-grained, same structure
    │       ├── Requirements.md
    │       ├── Spec.md
    │       ├── Plan-and-Progress.md
    │       └── Change-Log.md
    └── Deliverables/                # each finished feature delivers here
        └── <feature-module>/
            ├── Feature-Doc.md
            └── Principle-Doc.md
```

Minimum file requirements per level:

- **Top level (task-doc root)**: `STATE.md` (unique status entry) + `ENVIRONMENT.md` (user-written environment notes).
- **Overall directory (`Overall-<project>/`)**: Research, Requirements, Architecture, Plan-and-Progress, Change-Log—all five present; all feature modules and the Deliverables directory nested inside.
- **Feature-module level**: Requirements, Spec, Plan-and-Progress, Change-Log—all four present.
- **Submodule level**: Requirements, Spec, Plan-and-Progress, Change-Log—all four present.
- **Deliverables level**: Feature-Doc and Principle-Doc—both present.

## File Roles and Content Rules

### STATE.md (top-level status entry)

- **Role**: the first file the agent reads after session recovery or context compression, to rebuild "where am I, what's next" in the shortest time.
- **Content**: fixed four-field structure—current phase, current task, next action, known blockers; plus an index of the doc tree marking each module's completion status.

### ENVIRONMENT.md (root, user-written)

- **Role**: carries the user's description of the development environment for the agent to consult while coding and running; often contains sensitive information, so **never commit it to git**.
- **Content**: languages and versions, stack and frameworks, runtime environments and services, apikeys/credentials, proxies and ports, other constraints.

### Requirements.md

- **Role**: defines "what problem to solve, where the boundary is"—the anchor against goal drift.
- **Content**: background and motivation, goals to achieve, explicit out-of-scope, success criteria. One copy per level (overall / feature module / submodule), each describing the requirements of its own scope.

### Research.md (overall directory only)

- **Role**: records the current theory and practice surveyed before starting, grounding design and specs; avoids designing from outdated intuition.
- **Content**: sources and dates, adoptable practices, rejected options and why, direct impact on this project's design.

### Architecture.md (overall directory only)

- **Role**: describes the system's overall structure and inter-module relations, keeping the architecture sound and maintainable, preventing patches on patches.
- **Content**: module breakdown and responsibilities, data/control flow, key design decisions and their trade-offs, inter-module dependencies.

### Spec.md (feature-module and submodule levels)

- **Role**: mandatory pre-coding artifact—"no implementation until the spec is settled." The objective standard for judging whether the implementation is correct.
- **Content**: architecture, interface definitions, inputs/outputs/behavior, edge cases and error handling, and a testable acceptance checklist (each item numbered and objectively decidable, not subjective prose; item numbers are used to map tasks in Plan-and-Progress).

### Plan-and-Progress.md

- **Role**: provides progress anchors so the task can be losslessly resumed at any interruption point.
- **Scope**: covers the whole lifecycle—documentation, coding, delivery. Task breakdown must span all phases: documentation (research, requirements, architecture, spec), coding and testing, delivery (Feature-Doc and Principle-Doc); registering only coding tasks is forbidden.
- **Task ownership**: module-level and submodule-level plans register only tasks that can be completed and accepted within that module's own scope; cross-module tasks or tasks depending on later integration phases (e.g., front-end/back-end integration, system-level integration testing) are registered only in the overall directory's Plan-and-Progress—pushing them down to module-level plans is forbidden.
- **Content**: **mandatory table format**, consisting of two kinds of tables:
  - **Task table**: one task per row; fixed columns—Task ID, Description, Files, Status, Acceptance-checklist mapping (enter the spec checklist item numbers this task maps to; if the mapped checklist items are complete, mark it in this column).
  - **Subtask table** (required when a task has subtasks): identical columns plus one more—Parent Task ID.
- **Status rules**: exactly four values—Not Started, In Progress, Done, Blocked. Task descriptions must be decidable now; future-pointing vague wording like "confirm at the integration phase" is forbidden. A task whose completion depends on factors outside this unit does not belong in this unit's plan; if it must be registered, its status can only be Blocked with the blocking reason and unblocking condition noted—never In Progress.
- **Plan zero-out**: before this directory counts as complete, every task in its plan must be resolved to Done or Blocked; all Blocked tasks must be synchronously registered in the parent (overall) plan, which then takes over tracking. Dangling statuses, or lingering In-Progress tasks carried into the next unit, are forbidden.

### Change-Log.md

- **Role**: change ledger for every document at each level (except Plan-and-Progress), keeping documents replayable and their latest version determinable; silent overwrites are forbidden.
- **Content**: registered entry by entry; each entry must carry four fields—date (to the second), affected document and section, before/after comparison, reason for the change.

### Feature-Doc.md (deliverables level)

- **Role**: tells users what the deliverable does and how to use it.
- **Content**: feature overview, usage steps and examples, configuration, known limitations.

### Principle-Doc.md (deliverables level)

- **Role**: explains "on what principles it was built, how the whole chain works," for later maintenance and retrospection.
- **Content**: implementation principles, end-to-end walkthrough, mapping to design/spec, trade-off notes.

## Anti-Compression Reread Protocol

At the following three moments, you **must** first reread the relevant status and spec files before continuing; proceeding on compressed residual memory is forbidden:

- **Before starting a new unit**: read that unit's spec file and the top-level status file.
- **After finishing a subtask**: update and reread the top-level status file.
- **After context compression**: reread the top-level status file and the current unit's spec file, and rebuild your working position from them.

## Commits and Checkpoints

- Commit once per finished work unit.
- Commit messages should reference the related doc paths so that code state and doc state correspond one-to-one, avoiding the drift of "code committed but docs not synced" or the reverse.

## Notes

1. **Sub-agent concurrency safety**: sub-agents may be used for parallel work, but concurrent competing writes to the same document are forbidden. At any moment a document has exactly one writer; otherwise updates get lost or corrupted. When dividing parallel tasks, isolate write boundaries by document/directory.
2. **Spec–implementation as a two-way contract**: when implementation reveals a mismatch with the spec, first write it back to the Change-Log, then continue coding; silently following a stale spec or privately deviating from it is forbidden. The spec is not one-way input but a two-way contract: implementation follows the spec; when the spec proves wrong or a better solution exists, first update the spec and log the change, then adjust the implementation accordingly.
3. **Environment doc never in git**: `ENVIRONMENT.md` often contains apikeys and credentials; it must be excluded from version control (added to `.gitignore`) and never committed.
4. **Incremental changes must be logged**: all incremental changes must be written to the Change-Log; the sole exception is Plan-and-Progress—it may be freely overwritten in place without logging.
