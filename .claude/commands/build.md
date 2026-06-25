---
description: Implement tasks incrementally — build, test, verify, commit. Add "auto" to run the whole plan in one approved pass.
---

Invoke the agent-skills:incremental-implementation skill alongside agent-skills:test-driven-development.

## Modes

- **`/build <task-issue-number>`** — implement a specific task issue, then stop.
- **`/build auto <spec-issue-number>`** — get a single approval, then implement *every* open task for that spec without stopping between them.
- **`/build`** — resume the task for the current branch (only valid when already on a task branch).

`$ARGUMENTS` selects the mode:
- A bare integer → single-task mode for that task issue.
- `auto <N>` or `<N> auto` → autonomous mode for spec issue N.
- Empty → resume mode: extract the task issue number from the current branch name (`feature/<NNNNN>-task-slug` → strip leading zeros). If the current branch is not a task branch, stop and ask the user to provide a task issue number.

## Default: one task

1. Fetch the task issue: `gh issue view <task-N> --json title,body`
2. Extract the task name from the title and the full task definition from the body
3. Load relevant context (existing code, patterns, types)
4. **Check out a task branch from main.** Switch to main, pull latest, then check out `feature/<NNNNN>-task-slug` where `NNNNN` is the task issue number zero-padded to five digits (e.g. `feature/00043-task-s3-lake-module`). If already on that branch (resume mode), skip this step.
5. Write a failing test for the expected behavior (RED)
6. Implement the minimum code to pass the test (GREEN)
7. Run the full test suite to check for regressions
8. Run the build to verify compilation
9. Commit with a descriptive message.
10. **Open a PR** from `feature/<NNNNN>-task-slug` to main. Run `gh pr create --title "task: <task-name>" --body "Closes #<task-N>\n\n<summary>"`. The issue will be automatically closed when the PR is merged. If rework is needed after review, continue on the same branch and push updates.
11. Stop

## Autonomous: the whole plan (`/build auto <spec-N>`)

Use this once a spec has been planned and you want to build every task in one approved pass. It removes manual stepping between tasks — **not** the verification. Every task still earns a passing test, its own commit, and its own PR from its feature branch to main.

1. **Fetch the spec.** Run `gh issue view <spec-N> --json title,body` to confirm the spec exists.
2. **Establish a clean baseline.** Run `git status --porcelain`. If there are uncommitted changes, stop and ask the user to commit or stash them.
3. **List all task issues.** Run `gh issue list --search "\"Spec: #<spec-N>\"" --json number,title,state --limit 100` to see overall progress.
4. **Require a plan.** If no task issues exist, stop and tell the user to run `/plan <spec-N>` first — do not invent requirements.
5. **Single checkpoint.** Present the full task list (open and closed) and wait for an unambiguous affirmative (e.g. "approve", "go", "yes"). Treat hedged responses ("looks reasonable", "I guess") as **not** approved. This is the only human gate.
6. **Execute every open task in dependency order.** For each open task issue (ordered by issue number), run the full default loop above (RED → GREEN → regression → build → commit → open PR from feature branch to main). Each task branch is cut from main. Stage only the files that task touched — never `git add -A` blindly.
7. **Stop and ask the user** (do not push through) when:
   - A test can't be made to pass or the build breaks without an obvious fix → follow agent-skills:debugging-and-error-recovery
   - The spec or task is ambiguous and needs a decision not covered by the issue
   - A task is high-risk or irreversible — auth/permission changes, destructive data migrations, payments, deletions, deploys, anything touching secrets, **or anything you can't undo with `git revert`** → follow agent-skills:doubt-driven-development and get explicit sign-off before continuing

   After the user resolves a blocker, they re-invoke `/build auto <spec-N>` — it resumes from the next open task issue.
8. **Summarize at the end:** tasks completed, tests added, PRs opened, and anything skipped, flagged, or left for the user.

If any step fails, follow the agent-skills:debugging-and-error-recovery skill.
