---
description: Break work into small verifiable tasks with acceptance criteria and dependency ordering
---

Invoke the agent-skills:planning-and-task-breakdown skill.

**Resolve the spec issue.** The spec issue number is in `$ARGUMENTS`. If no number was provided, ask the user: "Which spec issue should I plan? Run `/plan <issue-number>`." Fetch the spec with `gh issue view <N> --json title,body`.

Read the spec content and the relevant codebase sections. Then:

1. Enter plan mode — read only, no code changes
2. Identify the dependency graph between components
3. Slice work vertically (one complete path per task, not horizontal layers)
4. Write tasks with acceptance criteria and verification steps
5. Add checkpoints between phases
6. Present the plan for human review

When ready to save:

1. **Post the overall plan as a comment on the spec issue.** Run `gh issue comment <N> --body "## Plan\n\n<plan-markdown>"`.
2. **Create one GitHub issue per task.** For each task run `gh issue create --title "task: <task-name>" --body "<task-definition>\n\n---\nSpec: #<N>"`. Capture each task's issue number and zero-pad it to five digits.
3. **Link all tasks back to the spec.** Post one comment on the spec issue listing every task: `gh issue comment <N> --body "## Tasks\n\n- [ ] #<T1> <task-1-slug>\n- [ ] #<T2> <task-2-slug>\n..."`.
4. **Stop.** Do not run `/build` until the spec issue and task list have been reviewed. Implementation begins when the user approves and runs `/build <task-issue-number>`.
