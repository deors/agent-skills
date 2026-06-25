---
description: Assess production readiness of a release candidate — runbooks, observability, and security monitoring. Does not deploy code; deployment is done exclusively by CD workflows.
---

Invoke the agent-skills:shipping-and-launch skill alongside agent-skills:observability-and-instrumentation and agent-skills:security-and-hardening.

## What this command does — and does not do

`/ship` **does not deploy code.** Deployment to any environment is done exclusively through CD workflows triggered by CI after a merge. This command exists to ensure the team is ready to support and operate the change once it is live.

Run `/ship` when a release candidate artifact has been produced by CI and validated in staging (steps 15–19 of the lifecycle), before the PM/PO makes the production deployment decision (step 20).

## Resolving the release context

`$ARGUMENTS` may be a spec issue number, a PR number, or a release tag. Use it to scope what is being assessed.

1. Run `gh issue view <spec-N> --json title,body` to load the spec and understand the intended behavior.
2. Run `gh pr list --search "Spec: #<spec-N>" --state merged --json number,title` to list all merged PRs in this release.
3. Confirm the artifact has been deployed to staging and CT results are available before proceeding. If staging validation has not run, stop and tell the engineer to wait for it.

## Phase A — Parallel fan-out

Spawn three subagents concurrently using the Agent tool. Issue all three calls in a single assistant turn.

1. **`observability-auditor`** (use `agent-skills:observability-and-instrumentation`):
   - Are structured logs emitted for the new behavior? Are they queryable?
   - Are RED metrics (Rate, Errors, Duration) instrumented for every new service path?
   - Are distributed traces configured end-to-end?
   - Are dashboards created or updated to surface the new behavior?
   - Are symptom-based alerts defined? Do they have runbook links?

2. **`security-auditor`** (use the `security-auditor` persona):
   - Is runtime security monitoring configured for the new attack surface?
   - Are secrets rotated if the change touched credential handling?
   - Are access controls verified in the staging environment?
   - Are dependency CVEs re-checked against the release artifact?
   - Are audit logs enabled for any new sensitive operations?

3. **`operations-auditor`** (use `agent-skills:documentation-and-adrs`):
   - Does a runbook exist for operating and troubleshooting this change in production?
   - Are rollback procedures documented with exact steps and trigger conditions?
   - Are environment-specific configuration differences documented?
   - Are any manual pre- or post-deployment steps identified and assigned?
   - Are feature flags (if used) documented with their expected lifecycle and removal date?

## Phase B — Synthesis

Once all three reports are back, synthesize them into a production readiness report:

```markdown
## Production Readiness Report

**Release:** [spec title / release tag]
**Staging validation:** [CT results summary — passed / failed / partial]

### Readiness verdict: READY | NOT READY

### Blockers (must resolve before production deployment)
- [Source: finding + action required]

### Recommended actions (resolve before or immediately after deployment)
- [Source: finding + action required]

### Runbook
- Location: [link or file path]
- Rollback trigger: [conditions that would prompt rollback]
- Rollback procedure: [exact steps]
- Recovery time objective: [target]

### Observability
- Dashboards: [ready / link]
- Alerts: [configured / not configured — details]
- Log queries: [ready / link]

### Security monitoring
- Runtime monitoring: [configured / not configured — details]
- Access controls verified in staging: [yes / no]
- CVE check: [clean / findings]

### Operations checklist
- [ ] Runbook created or updated
- [ ] Rollback steps verified in staging
- [ ] Feature flags documented (if applicable)
- [ ] On-call team notified
```

## Rules

1. Do not run this command before staging validation (CT) has completed. Staging is the last gate before production — it must pass.
2. The three Phase A subagents run in parallel — never sequentially.
3. A NOT READY verdict blocks the PM/PO production deployment decision. The team must resolve blockers and re-run `/ship` before proceeding.
4. Runbook creation is mandatory, not optional. If a runbook does not exist, create it as part of this command.
5. This command produces a readiness report — the PM/PO uses it to decide when to trigger the CD workflow for production. The command never triggers that workflow itself.
