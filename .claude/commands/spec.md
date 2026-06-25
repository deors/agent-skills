---
description: Start spec-driven development — write a structured specification before writing code
---

Invoke the agent-skills:spec-driven-development skill.

Begin by understanding what the user wants to build. Ask clarifying questions about:

1. The objective and target users
2. Core features and acceptance criteria
3. Tech stack preferences and constraints
4. Known boundaries (what to always do, ask first about, and never do)

Then generate a structured spec covering all six core areas: objective, commands, project structure, code style, testing strategy, and boundaries. For project structure, always use tree characters (├──, └──, │) to show directory hierarchy, rendered as a 4-space-indented block (not triple backticks) for reliable GitHub rendering.

When ready to save:

1. **Create the GitHub issue.** Run `gh issue create --title "spec: <slug>" --body "<spec-markdown>"` and capture the issue number from the returned URL (e.g. `https://github.com/owner/repo/issues/42` → `42`). Zero-pad it to five digits (e.g. `00042`).

2. **Confirm with the user.** Present the spec issue URL. Tell the user to run `/plan <issue-number>` next to break the spec into tasks before writing any code.
