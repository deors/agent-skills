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

- Read the spec issue and relevant codebase sections
- Identify existing patterns and conventions
- Map dependencies between components
- Note risks and unknowns

**Do NOT write code during planning.** The output is a plan posted to the spec issue and a set of task issues, not implementation.

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

Each task becomes a GitHub issue with title `task: <slug>` and body following this structure:

```markdown
[One paragraph explaining what this task accomplishes.]

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
**Dependencies:** #<T1>, #<T2> (other task issue numbers), or None

---
Spec: #<N>
```

The `Spec: #<N>` line at the bottom links the task back to its parent spec issue and is used by `/build` to discover all tasks for a given spec.

### Step 5: Order and Checkpoint

Arrange tasks so that:

1. Dependencies are satisfied (build foundation first)
2. Each task leaves the system in a working state
3. Verification checkpoints occur after every 2-3 tasks
4. High-risk tasks are early (fail fast)

Add explicit checkpoints in the plan comment using issue references:

```markdown
## Checkpoint: After #T1, #T2, #T3
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

## Output: GitHub Issues

All planning output lives in GitHub issues — no local files are created.

**Plan comment:** Post the overall architecture, dependency graph, task table, checkpoints, risks, and open questions as a comment on the spec issue using `gh issue comment <spec-N> --body "<plan-markdown>"`.

**Task issues:** Create one GitHub issue per task using `gh issue create --title "task: <slug>" --body "<task-body>"`. Each task issue body must end with `Spec: #<spec-N>` so `/build` can discover tasks by spec.

**Task list comment:** After all task issues are created, post a second comment on the spec issue that lists every task with its issue number. This gives reviewers a single place to see the full scope:

```markdown
## Tasks

### Phase 1 — Foundation

- [ ] #T1 task-slug

**Checkpoint:** [gate condition]. Human review before Phase 2.

### Phase 2 — Core

- [ ] #T2 task-slug
- [ ] #T3 task-slug *(parallelizable with #T2)*

**Checkpoint:** [gate condition]. Human review before Phase 3.
```

## Plan Comment Template

```markdown
## Plan

### Overview
[One paragraph summary of what we're building]

Spec: #<N>

### Dependency Graph
[ASCII diagram showing component dependencies]

### Architecture Decisions
- [Key decision 1 and rationale]
- [Key decision 2 and rationale]

### Tasks

| # | Issue | Phase | Depends on | Parallelizable |
|---|-------|-------|------------|----------------|
| 1 | #T1 Short title | 1 — Foundation | — | No |
| 2 | #T2 Short title | 2 — Core | #T1 | No |
| 3 | #T3 Short title | 3 — Polish | #T2 | Yes |

### Checkpoints

| After | Gate |
|-------|------|
| #T1 | [what must be true before proceeding] |
| #T2, #T3 | [what must be true before proceeding] |

### Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk] | High / Med / Low | [Strategy] |

### Open Questions
- [Question needing human input]
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
- No verification steps in any task
- All tasks are XL-sized
- No checkpoints between tasks
- Dependency order isn't considered
- Task issues missing the `Spec: #<N>` line (breaks `/build` task discovery)
- Task bodies written without acceptance criteria or verification steps

## Verification

Before starting implementation, confirm:

- [ ] A plan comment has been posted on the spec issue
- [ ] Every task has its own GitHub issue with title `task: <slug>`
- [ ] Every task issue body ends with `Spec: #<spec-N>`
- [ ] Every task has acceptance criteria
- [ ] Every task has a verification step
- [ ] Task dependencies reference other tasks by issue number (`#<T>`)
- [ ] No task touches more than ~5 files
- [ ] Checkpoints exist between major phases
- [ ] A task list comment has been posted on the spec issue linking all task issues
- [ ] The human has reviewed and approved the plan before implementation begins
