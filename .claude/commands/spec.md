---
description: Start spec-driven development — write a structured specification before writing code
---

Invoke the agent-skills:spec-driven-development skill.

Begin by understanding what the user wants to build. Ask clarifying questions about:

1. The objective and target users
2. Core features and acceptance criteria
3. Tech stack preferences and constraints
4. Known boundaries (what to always do, ask first about, and never do)

Then generate a structured spec covering all six core areas: objective, commands, project structure, code style, testing strategy, and boundaries.

When ready to save:

1. **Determine the sequence number.** List `specs/` to find the highest existing `NNN` prefix. Increment by one (zero-padded to three digits) for the new spec.
2. **Create the spec branch.** Check out a new branch: `feature/spec-NNN-slug` (e.g. `feature/spec-001-add-login-window`). Do not write the spec on main.
3. **Save the spec.** Write `specs/NNN-slug.md` on that branch. Do not use `SPEC.md` at the project root.
4. **Commit.** Stage only the spec file and commit it to `feature/spec-NNN-slug`.
5. **Confirm with the user.** Present the spec for review. Tell the user to run `/plan` next — on the same branch — before opening a PR.
