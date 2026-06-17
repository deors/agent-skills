---
description: Break work into small verifiable tasks with acceptance criteria and dependency ordering
---

Invoke the agent-skills:planning-and-task-breakdown skill.

Read the existing spec (`specs/NNN-slug.md`) and the relevant codebase sections. Then:

1. Enter plan mode — read only, no code changes
2. Identify the dependency graph between components
3. Slice work vertically (one complete path per task, not horizontal layers)
4. Write tasks with acceptance criteria and verification steps
5. Add checkpoints between phases
6. Present the plan for human review

When ready to save:

1. **Confirm the branch.** Check that the current branch is `feature/spec-NNN-slug` (created by `/spec`). If not, check it out — do not commit planning artifacts to main or any other branch.
2. **Save all planning artifacts.** Write `tasks/NNN-slug/plan.md`, `tasks/NNN-slug/todo.md`, and one `tasks/NNN-slug/NNN-NN-slug.md` file per task.
3. **Commit.** Stage all files under `tasks/NNN-slug/` and commit them to `feature/spec-NNN-slug`.
4. **Open or update the PR.** The PR from `feature/spec-NNN-slug` → main should contain the spec and all task files together, so reviewers see the full scope in one place.
5. **Stop.** Do not run `/build` until the PR is approved and merged. Implementation branches are cut from main after the spec branch lands.
