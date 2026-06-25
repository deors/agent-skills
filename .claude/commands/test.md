---
description: Two intents — check the test state of a task branch, or write missing tests for existing code using TDD.
---

Invoke the agent-skills:test-driven-development skill.

`/test` is a human-invoked command. It is never called automatically. `/build` handles its own TDD cycle internally; `/test` exists for the two situations where the engineer needs to act on tests directly.

## Intent 1 — Check the state of a task

Use this to inspect the test results for work the agent has produced, validate a branch before requesting a review, or investigate a failure the agent surfaced.

**`/test`** — run the full test suite on the current branch and report results.

**`/test <task-issue-number>`** — switch to `feature/<NNNNN>-slug` for that task issue, run the full test suite, then report results. Use when you want to check a task without being on its branch already.

### Process

1. Identify the target branch (current branch or `feature/<NNNNN>-slug` from the task issue number)
2. Run the full test suite
3. Run the build to check compilation
4. Report results clearly: tests passed, tests failed (with failure details), tests skipped, and whether the build is clean
5. For any failure, identify the failing test, the assertion that failed, and the likely cause — do not just print the stack trace
6. If browser-related code is under test, also invoke agent-skills:browser-testing-with-devtools for runtime verification

Do not fix failures as part of this intent. Report them so the engineer can decide whether to fix manually, return the task to the agent via `/build`, or flag the issue.

## Intent 2 — Write missing tests for existing code

Use this when existing code has no tests, or test coverage is known to be inadequate, and the engineer wants the agent to produce them using TDD.

**`/test <task-issue-number> --add-tests`** or **`/test --add-tests`** — analyse the target code for missing test coverage and write tests for it.

### Steps

1. Identify the target code — from the task issue body, or from the current branch if no issue is given
2. Read the existing code to understand its intended behavior, edge cases, and failure paths
3. Identify what is not covered: missing happy-path tests, missing edge cases, missing error path tests
4. For each gap, follow the TDD cycle (invoke `agent-skills:test-driven-development`):
   - Write the test first, describing the expected behavior — it must be written against the existing code without modifying it
   - Confirm the test passes against the existing implementation (these are not failing tests — the code already exists; the tests are the missing artifact)
   - If a test reveals a defect in the existing code, surface it as a finding rather than fixing the code silently
5. Run the full test suite to confirm no regressions were introduced by the new tests
6. Report: tests added, coverage gaps remaining, and any defects surfaced

Do not modify existing implementation code as part of this intent. If a defect is found, report it so the engineer can open a new task issue to address it.
