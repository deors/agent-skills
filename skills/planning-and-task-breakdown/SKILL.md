---
name: planning-and-task-breakdown
description: Breaks work into ordered tasks. Use when you have a spec or clear requirements and need to break work into implementable tasks. Use when a task feels too large to start, when you need to estimate scope, or when parallel work is possible.
---

# Planning and Task Breakdown

## Overview

Decompose work into small, verifiable tasks with explicit acceptance criteria. Good task breakdown is the difference between an agent that completes work reliably and one that produces a tangled mess. Every task should be small enough to implement, test, and verify in a single focused session.

## When to Use

- You have a spec and need to break it into implementable units
- A task feels too large or vague to start
- Work needs to be parallelized across multiple agents or sessions
- You need to communicate scope to a human
- The implementation order isn't obvious

**When NOT to use:** Single-file changes with obvious scope, or when the spec already contains well-defined tasks.

## The Planning Process

### Step 1: Enter Plan Mode

Before writing any code, operate in read-only mode:

- Read the spec and relevant codebase sections
- Identify existing patterns and conventions
- Map dependencies between components
- Note risks and unknowns

**Do NOT write code during planning.** The output is a plan document, not implementation.

### Step 2: Identify the Dependency Graph

Map what depends on what:

```
Database schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

Implementation order follows the dependency graph bottom-up: build foundations first.

### Step 3: Slice Vertically

Instead of building all the database, then all the API, then all the UI — build one complete feature path at a time:

**Bad (horizontal slicing):**
```
Task 1: Build entire database schema
Task 2: Build all API endpoints
Task 3: Build all UI components
Task 4: Connect everything
```

**Good (vertical slicing):**
```
Task 1: User can create an account (schema + API + UI for registration)
Task 2: User can log in (auth schema + API + UI for login)
Task 3: User can create a task (task schema + API + UI for creation)
Task 4: User can view task list (query + API + UI for list view)
```

Each vertical slice delivers working, testable functionality.

### Step 4: Write Tasks

Each task lives in its own file named `NNN-NN-slug.md` inside the spec's task directory (e.g. `tasks/001-add-login-window/001-02-api-layer.md`). Never embed task details inline in `plan.md`.

Each task file follows this structure:

```markdown
# Task NNN-NN — Short descriptive title

**Branch:** `feature/task-NNN-NN-short-slug`
**Phase:** [phase number and name]
**Dependencies:** [NNN-NN-slug](NNN-NN-slug.md), or None

## Description

One paragraph explaining what this task accomplishes.

## Acceptance Criteria

- [ ] [Specific, testable condition]
- [ ] [Specific, testable condition]

## Verification

- [ ] Tests pass: `npm test -- --grep "feature-name"`
- [ ] Build succeeds: `npm run build`
- [ ] Manual check: [description of what to verify]

## Files

- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Estimated scope:** [Small: 1-2 files | Medium: 3-5 files | Large: 5+ files]
```

### Step 5: Order and Checkpoint

Arrange tasks so that:

1. Dependencies are satisfied (build foundation first)
2. Each task leaves the system in a working state
3. Verification checkpoints occur after every 2-3 tasks
4. High-risk tasks are early (fail fast)

Add explicit checkpoints using task IDs, not ordinal numbers:

```markdown
## Checkpoint: After NNN-01, NNN-02, NNN-03
- [ ] All tests pass
- [ ] Application builds without errors
- [ ] Core user flow works end-to-end
- [ ] Review with human before proceeding
```

## Task Sizing Guidelines

| Size | Files | Scope | Example |
|------|-------|-------|---------|
| **XS** | 1 | Single function or config change | Add a validation rule |
| **S** | 1-2 | One component or endpoint | Add a new API endpoint |
| **M** | 3-5 | One feature slice | User registration flow |
| **L** | 5-8 | Multi-component feature | Search with filtering and pagination |
| **XL** | 8+ | **Too large — break it down further** | — |

If a task is L or larger, it should be broken into smaller tasks. An agent performs best on S and M tasks.

**When to break a task down further:**
- It would take more than one focused session (roughly 2+ hours of agent work)
- You cannot describe the acceptance criteria in 3 or fewer bullet points
- It touches two or more independent subsystems (e.g., auth and billing)
- You find yourself writing "and" in the task title (a sign it is two tasks)

## Output File Location

Save files under a subdirectory of `tasks/` named after the spec being implemented, using the same zero-padded three-digit numeric prefix. Each task gets its own file named `NNN-NN-slug.md` where `NNN` is the spec number and `NN` is the task sequence within that spec (both zero-padded). Example: if the spec is `specs/001-add-login-window.md` and has three tasks:

```
tasks/001-add-login-window/plan.md              ← architecture, dependency graph, risks; links to task files
tasks/001-add-login-window/todo.md              ← flat checklist with links to task files
tasks/001-add-login-window/001-01-scaffold.md   ← full task detail: description, acceptance criteria, verification, files
tasks/001-add-login-window/001-02-api-layer.md
tasks/001-add-login-window/001-03-ui-layer.md
```

The `NNN-NN` prefix encodes spec ownership in the filename. This eliminates branch-name collisions when multiple people work tasks from different specs in parallel — `feature/task-001-02-api-layer` and `feature/task-002-02-api-layer` are unambiguously distinct.

`plan.md` contains the architecture overview, dependency graph, and risks — not individual task details. Individual task details live exclusively in their own numbered files. `todo.md` is a flat checklist that links to each task file.

Never save `plan.md` or `todo.md` directly into the `tasks/` root — multiple specs accumulate over the life of a project and a flat structure does not scale.

**Branch:** All planning artifacts — `plan.md`, `todo.md`, and every `NNN-NN-slug.md` file — must be committed to the same `feature/spec-NNN-slug` branch as the spec itself. Do not commit planning work to main or to an implementation branch. After all task files are written and committed, open (or update) the PR from `feature/spec-NNN-slug` → main so the full spec + task breakdown can be peer reviewed together before implementation begins.

## plan.md Template

`plan.md` holds architecture and navigation only — no inline task details. Task details live in individual `NNN-NN-slug.md` files.

```markdown
# Implementation Plan: [Feature/Project Name]

## Overview
[One paragraph summary of what we're building]

Spec: [`specs/NNN-slug.md`](../../specs/NNN-slug.md)

## Dependency Graph
[ASCII diagram showing component dependencies]

## Architecture Decisions
- [Key decision 1 and rationale]
- [Key decision 2 and rationale]

## Tasks

| # | File | Phase | Depends on | Parallelizable |
| --- | --- | --- | --- | --- |
| [NNN-01](NNN-01-slug.md) | Short title | 1 — Foundation | — | No |
| [NNN-02](NNN-02-slug.md) | Short title | 2 — Core | NNN-01 | No |
| [NNN-03](NNN-03-slug.md) | Short title | 3 — Polish | NNN-02 | Yes |

## Checkpoints

| After task(s) | Gate |
| --- | --- |
| NNN-01 | [what must be true before proceeding] |
| NNN-02, NNN-03 | [what must be true before proceeding] |

## Risks and Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| [Risk] | High / Med / Low | [Strategy] |

## Open Questions
- [Question needing human input]
```

## todo.md Template

`todo.md` is a flat checklist that links to each task file — the single place a developer looks to find the next pending task.

```markdown
# Todo: [Feature/Project Name]

Full plan: [plan.md](plan.md) | Spec: [specs/NNN-slug.md](../../specs/NNN-slug.md)

## Phase 1 — Foundation

- [ ] [NNN-01 — Task title](NNN-01-slug.md)

**Checkpoint:** [gate condition]. Human review before Phase 2.

## Phase 2 — Core

- [ ] [NNN-02 — Task title](NNN-02-slug.md)
- [ ] [NNN-03 — Task title](NNN-03-slug.md) *(parallelizable with NNN-02)*

**Checkpoint:** [gate condition]. Human review before Phase 3.
```

## Parallelization Opportunities

When multiple agents or sessions are available:

- **Safe to parallelize:** Independent feature slices, tests for already-implemented features, documentation
- **Must be sequential:** Database migrations, shared state changes, dependency chains
- **Needs coordination:** Features that share an API contract (define the contract first, then parallelize)

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll figure it out as I go" | That's how you end up with a tangled mess and rework. 10 minutes of planning saves hours. |
| "The tasks are obvious" | Write them down anyway. Explicit tasks surface hidden dependencies and forgotten edge cases. |
| "Planning is overhead" | Planning is the task. Implementation without a plan is just typing. |
| "I can hold it all in my head" | Context windows are finite. Written plans survive session boundaries and compaction. |

## Red Flags

- Starting implementation without a written task list
- Tasks that say "implement the feature" without acceptance criteria
- No verification steps in the plan
- All tasks are XL-sized
- No checkpoints between tasks
- Dependency order isn't considered
- Task details written inline in `plan.md` instead of individual `NNN-NN-slug.md` files
- Task files named without the `NNN-NN` prefix (breaks parallel branch isolation across specs)
- `plan.md` or `todo.md` saved directly in `tasks/` root instead of `tasks/NNN-slug/`

## Verification

Before starting implementation, confirm:

- [ ] All files are on the `feature/spec-NNN-slug` branch (not main, not an implementation branch)
- [ ] Every task lives in its own `NNN-NN-slug.md` file under `tasks/NNN-slug/`
- [ ] Every task file has the correct `NNN-NN` prefix matching its spec number
- [ ] Every task file includes a `**Branch:**` line with `feature/task-NNN-NN-slug`
- [ ] Every task has acceptance criteria
- [ ] Every task has a verification step
- [ ] Task dependencies reference other tasks by `NNN-NN-slug` ID, not by ordinal ("Task 2")
- [ ] No task touches more than ~5 files
- [ ] Checkpoints exist between major phases and reference tasks by `NNN-NN` ID
- [ ] `plan.md` links to task files — no inline task details
- [ ] `todo.md` is a flat checklist linking to all task files
- [ ] All planning artifacts are committed to `feature/spec-NNN-slug`
- [ ] A PR from `feature/spec-NNN-slug` → main is open for peer review
- [ ] The human has reviewed and approved the plan before implementation begins
