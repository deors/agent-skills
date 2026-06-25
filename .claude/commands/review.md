---
description: Conduct a five-axis code review on the task branch and post the result as a comment on the open PR.
---

Invoke the agent-skills:code-review-and-quality skill.

## Resolving the branch and PR

`$ARGUMENTS` may be a task issue number, a PR number, or empty.

1. **If a task issue number is given:** derive the branch name as `feature/<NNNNN>-slug` (zero-pad the number to five digits). Run `gh pr list --head feature/<NNNNN>-slug --json number,url` to find the open PR. Checkout that branch before reviewing.
2. **If a PR number is given:** run `gh pr view <PR> --json headRefName,url` to get the branch name. Checkout that branch.
3. **If no argument:** use the current branch. Run `gh pr list --head $(git branch --show-current) --json number,url` to find its PR. If there is no open PR, stop and tell the engineer to open one first.

Do not review `main` or any branch that does not have an open PR — there is nothing to post the result to.

## Review axes

Run the review against the diff between the task branch and `main` across all five axes:

1. **Correctness** — Does it match the task spec (fetch from `gh issue view <task-N> --json body`)? Edge cases handled? Tests adequate and actually verifying the right behavior?
2. **Readability** — Clear names? Straightforward logic? No unnecessarily deep nesting?
3. **Architecture** — Follows existing patterns? Clean boundaries? Right abstraction level? No circular dependencies?
4. **Security** — Input validated at boundaries? Secrets out of code and logs? Auth checked where needed? (Invoke agent-skills:security-and-hardening for depth.)
5. **Performance** — No N+1 queries? No unbounded operations? No unnecessary synchronous blocking? (Invoke agent-skills:performance-optimization for depth.)

Categorize every finding:
- **Critical** — must fix before merge (security vulnerability, data loss risk, broken functionality)
- **Important** — should fix before merge (missing test coverage, wrong abstraction, poor error handling)
- **Suggestion** — consider for improvement (naming, style, optional optimization)

## Posting the result

Once the review is complete, post it as a comment on the PR:

```bash
gh pr comment <PR-number> --body "<review-markdown>"
```

The review comment must follow this template:

```markdown
## Review

**Verdict:** APPROVE | REQUEST CHANGES

**Overview:** [1–2 sentences summarizing the change and overall assessment]

### Critical Issues
- `file:line` — description and recommended fix

### Important Issues
- `file:line` — description and recommended fix

### Suggestions
- `file:line` — description

### What's Done Well
- [at least one specific positive observation]

### Verification
- Tests reviewed: [yes/no + observations]
- Build verified: [yes/no]
- Security checked: [yes/no + observations]
```

Do not output the review only to the terminal. The PR comment is the required artifact. Confirm the comment URL after posting.
