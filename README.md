# Agent Skills — Issue-Centric SDLC

**Production-grade engineering skills for AI coding agents, built around GitHub as the single source of truth.**

Every artifact in this lifecycle — specs, tasks, changes, reviews — lives in GitHub. Slash commands are the interface between engineers and AI agents. GitHub Actions validate every change, whether human-authored or AI-authored, before it can become a releasable artifact.

```
  DEFINE         PLAN          BUILD        VERIFY        REVIEW         SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Spec │ ───▶ │Tasks │ ───▶ │ Code │ ───▶ │Tests │ ───▶ │  PR  │ ───▶ │  CD  │
 │Issue │      │Issues│      │  PR  │      │  CI  │      │Merge │      │Deploy│
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec         /plan         /build        /test        /review        /ship
```

---

## The Lifecycle

The end-to-end flow runs in two halves. The first half (steps 1–14) takes a feature from idea to merged PR. The second half (steps 15–20) takes a merged change from CI to a releasable artifact in production.

### Half 1 — Spec to Merged PR

| Step | Action | Actor | Command | GitHub Artifact |
|------|--------|-------|---------|-----------------|
| 1 | Engineer writes spec with AI assistance | Human + AI | `/spec` | Issue: `spec: <slug>` |
| 2 | Spec is saved to GitHub | AI | automatic | Issue created, URL returned |
| 3 | Engineer breaks spec into tasks | Human + AI | `/plan <spec-N>` | Comment on spec issue + task issues created |
| 4 | Plan is saved to GitHub | AI | automatic | Task issues linked to spec |
| 5 | Task issues added to backlog | AI | automatic | GitHub Project board updated |
| 6 | A task  is assigned to Agent (also possible to assign all tasks in a spec) | Human | `/build <task-N>` or `/build auto <spec-N>` | Branch: `feature/<NNNNN>-slug` |
| 7 | Agent writes tests and implements code using TDD (red→green→refactor) | AI | automatic or `/test` | Commits on feature branch |
| 8 | Agent validates no regressions before committing | AI | automatic | Commits on feature branch |
| 9 | Agent updates docs if needed | AI | automatic | Commit on feature branch |
| 10 | Agent opens a PR | AI | automatic | PR: `task: <task-name>` |
| 11 | CI checks the PR | CI | GitHub Actions | Status checks on PR |
| 12 | Agent reviews the PR | AI | automatic or `/review` | Review comment posted to PR |
| 13 | Dev team reviews the PR | Human | GitHub UI | Review approval |
| 14 | Dev team approves and merges | Human | GitHub UI | PR merged + task issue closed |

### Half 2 — Merged PR to Production

| Step | Action | Actor | Trigger | Output |
|------|--------|-------|---------|--------|
| 15 | CI checks merged code | CI | automatic on merge | Build + test run on main |
| 16 | CI produces a snapshot artifact | CI | automatic | Versioned artifact (container, package, binary) |
| 17 | CD deploys artifact to staging | CD | automatic | Deployment to staging environment |
| 18 | CT validates the artifact | CT | automatic | Integration / smoke test run |
| 19 | Artifact is flagged releasable (or not) | CT | automatic | Release tag or failure report |
| 20 | CD deploys artifact to production | PM/PO | PM/PO decision with inputs from `/ship` | Production deployment |

> GitHub and CI/CT/CD treats human-authored and AI-authored changes identically. Every change goes through the same harness — no exceptions, no shortcuts.

> Deploying a releasable artifact to production remains the PM/PO's decision. The harness codified in the CI/CT/CD workflows validates; the team decides when to ship.

---

## Commands

Seven slash commands map to the lifecycle. Each one activates the right skills automatically and persists its outputs to GitHub.

| Command | What it does | Saves to GitHub |
|---------|--------------|-----------------|
| `/spec` | AI-assisted spec writing — interviews the user, generates a structured PRD or ADR | Issue: `spec: <slug>` |
| `/plan <N>` | Reads spec issue `N`, breaks it into actionable tasks (carpaccio exercise), links them back to the originating spec | Task issues + comment on spec issue |
| `/build <T>` | Fetches task issue `T`, implements on a feature branch, opens a PR when ready, keeps Human in the loop in case of doubts or inability to finish the task | Branch + PR (closes `#T` on merge) |
| `/build auto <N>` | Builds every open task for spec `N` in one approved pass, still keeping Human in the loop if needed | One PR per task, all linked to spec |
| `/test` | Human-invoked only. Two intents: (1) run and report the test state of a task branch; (2) write missing tests for existing untested code using TDD. Never called automatically by `/build` | Intent 1: none (run only); Intent 2: commits with new tests on the branch |
| `/review` | Five-axis review (correctness, readability, architecture, security, performance) on the task branch diff — resolves the correct branch from a task issue number or PR number, posts the full review as a comment on the open PR | Review comment on PR |
| `/ship` | Production readiness assessment for a release candidate — does not deploy code. Verifies staging validation passed, checks observability (dashboards, alerts), security monitoring, and runbook completeness. Produces a readiness report for the PM/PO | Production readiness report comment |

### `/build auto` — autonomous mode

Once a spec is planned, `/build auto <spec-N>` builds every open task in a single approved pass. You approve once; the agent runs all tasks without stopping between them. Each task still earns a failing test, a passing test, its own commit, and its own PR. The agent pauses only on failures, ambiguity, or irreversible operations (auth changes, data migrations, deletions, secrets), effectively keeping the Human in the loop to solve those situations.

After a blocker is resolved, re-run `/build auto <spec-N>` — it resumes from where it called out the Human.

---

## Backlog and Kanban

The GitHub Project board is the single source of backlog truth.

- **Spec issues** are the epics — they anchor the plan and all task issues.
- **Task issues** are the unit of work. Each one has acceptance criteria and a verification step. They can be assigned to engineers or AI agents.
- **Column states** map to the lifecycle: `Todo` → `In Progress` → `In Review` → `Done`.
- Task issues **close automatically** when their PR is merged (via the `Closes #N` link in the PR body).

It remains the team's responsibility to keep the backlog complete, properly prioritized, and to minimize the work in progress. The agent creates issues and code; the team owns ideation, priority, and validation, and decision to ship.

---

## Two Modes of Operation

### Human-in-the-Loop

The engineer controls the pace. Each task is a separate decision point.

```
Engineer runs /build <task-N>
  → Agent implements on feature/<NNNNN>-slug
  → Agent opens PR
  → Engineer reviews and merges
  → Engineer runs /build <next-task-N>
```

### Human-out-of-the-Loop (Autopilot)

The engineer approves the plan once. The agent runs all tasks end to end.

```
Engineer runs /build auto <spec-N>
  → Agent presents full task list
  → Engineer approves ("go")
  → Agent implements every open task in order
    → For each: branch → TDD → commit → PR
  → Engineer reviews and merges each PR at their own pace
```

The only human gates in autopilot mode are the initial approval, explicit blockers (failures, ambiguity, high-risk operations), and final PR reviews. Verification — tests, build, regressions — still runs for every task.

---

## CI/CD with GitHub Actions

GitHub Actions are the enforcement layer for everything in Half 2.

**On every PR:**
- Build verification
- Full test suite (unit, integration, end-to-end)
- Static analysis and linting
- Security scanning (dependency CVEs, SAST)

**On merge to main:**
- Same checks run again on the merged commit
- A versioned snapshot artifact is produced
- CD deploys the artifact to staging
- Continuous testing validates the deployment
- The artifact is flagged `releasable` or `non-releasable` — non-releasable must be fixed with priority

**All agentic operations** (spec, plan, build, review) can be run from the IDE (Claude Code in VS Code), from the CLI (`claude`), or from a cloud agent harness (GitHub.com, custom-made app). The harness codified in the pipeline (key software engineering behaviors) is identical regardless of who or what authored the change.

---

## All 24 Skills

Commands are the entry points. The pack includes 24 skills — 23 lifecycle skills plus the `using-agent-skills` meta-skill. Skills activate automatically based on context; you can also invoke any skill directly.

### Meta

| Skill | Trigger |
|-------|---------|
| [using-agent-skills](skills/using-agent-skills/SKILL.md) | Starting a session or deciding which skill applies |

### Define

| Skill | Trigger |
|-------|---------|
| [interview-me](skills/interview-me/SKILL.md) | The ask is underspecified — extract what the user actually wants |
| [idea-refine](skills/idea-refine/SKILL.md) | A rough concept that needs structured exploration before spec |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Starting any new project, feature, or significant change |

### Plan

| Skill | Trigger |
|-------|---------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | A spec exists and needs to be decomposed into task issues |

### Build

| Skill | Trigger |
|-------|---------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Any change touching more than one file |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Implementing logic, fixing bugs, or changing behavior |
| [context-engineering](skills/context-engineering/SKILL.md) | Starting a session, switching tasks, or when output quality drops |
| [source-driven-development](skills/source-driven-development/SKILL.md) | Need authoritative, source-cited code for any framework or library |
| [doubt-driven-development](skills/doubt-driven-development/SKILL.md) | High stakes: production, security, irreversible operations |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | Building or modifying user-facing interfaces |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | Designing APIs, module boundaries, or public interfaces |

### Verify

| Skill | Trigger |
|-------|---------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Building or debugging anything that runs in a browser |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | Tests fail, builds break, or behavior is unexpected |

### Review

| Skill | Trigger |
|-------|---------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Before merging any change |
| [code-simplification](skills/code-simplification/SKILL.md) | Code works but is harder to read or maintain than it should be |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | User input, auth, data storage, or external integrations |
| [performance-optimization](skills/performance-optimization/SKILL.md) | Performance requirements exist or regressions are suspected |

### Ship

| Skill | Trigger |
|-------|---------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | Making any code change (always active) |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | Setting up or modifying build and deploy pipelines |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | Removing old systems, migrating users, sunsetting features |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | Architectural decisions, API changes, or shipping features |
| [observability-and-instrumentation](skills/observability-and-instrumentation/SKILL.md) | Adding telemetry or shipping anything that runs in production |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | Preparing to deploy to production |

---

## Agent Personas

Four specialist personas for targeted reviews. They are invoked in parallel by `/ship` and can be invoked individually via `/review`.

| Agent | Role | Invoked by |
|-------|------|------------|
| [code-reviewer](agents/code-reviewer.md) | Staff Engineer — five-axis review, "would a staff engineer approve this?" standard | `/review`, `/ship` |
| [test-engineer](agents/test-engineer.md) | QA Specialist — test strategy, coverage gaps, Prove-It pattern | `/ship` |
| [security-auditor](agents/security-auditor.md) | Security Engineer — OWASP Top 10, threat modeling, CVEs | `/ship` |
| [web-performance-auditor](agents/web-performance-auditor.md) | Performance Engineer — Core Web Vitals audit, profiling | `/webperf` |

Personas defined in `.claude/agents/` or `~/.claude/agents/` take precedence over the plugin versions. User-level definitions win by design.

---

## Reference Checklists

Quick-reference material that skills pull in when needed:

| Reference | Covers |
|-----------|--------|
| [testing-patterns.md](references/testing-patterns.md) | Test structure, naming, mocking, React/API/E2E examples, anti-patterns |
| [security-checklist.md](references/security-checklist.md) | Pre-commit checks, auth, input validation, headers, CORS, OWASP Top 10 |
| [performance-checklist.md](references/performance-checklist.md) | Core Web Vitals targets, frontend/backend checklists, measurement commands |
| [accessibility-checklist.md](references/accessibility-checklist.md) | Keyboard nav, screen readers, visual design, ARIA, testing tools |

---

## Project Structure

```text
root/
├── .claude/commands/                  # 8 slash commands (Claude Code)
├── skills/                            # 24 skills (23 lifecycle + 1 meta)
│   ├── interview-me/                  #   Define
│   ├── idea-refine/                   #   Define
│   ├── spec-driven-development/       #   Define
│   ├── planning-and-task-breakdown/   #   Plan
│   ├── incremental-implementation/    #   Build
│   ├── context-engineering/           #   Build
│   ├── source-driven-development/     #   Build
│   ├── doubt-driven-development/      #   Build
│   ├── frontend-ui-engineering/       #   Build
│   ├── test-driven-development/       #   Build
│   ├── api-and-interface-design/      #   Build
│   ├── browser-testing-with-devtools/ #   Verify
│   ├── debugging-and-error-recovery/  #   Verify
│   ├── code-review-and-quality/       #   Review
│   ├── code-simplification/           #   Review
│   ├── security-and-hardening/        #   Review
│   ├── performance-optimization/      #   Review
│   ├── git-workflow-and-versioning/   #   Ship
│   ├── ci-cd-and-automation/          #   Ship
│   ├── deprecation-and-migration/     #   Ship
│   ├── documentation-and-adrs/        #   Ship
│   ├── observability-and-instrumentation/ # Ship
│   ├── shipping-and-launch/           #   Ship
│   └── using-agent-skills/            #   Meta
├── agents/                            # 4 specialist personas
├── references/                        # 4 supplementary checklists
├── hooks/                             # Session lifecycle hooks
├── docs/                              # Setup and skill anatomy guides
├── AGENTS.md                          # Agent harness configuration
├── CLAUDE.md                          # Claude Code project instructions
└── plugin.json                        # Plugin manifest
```

---

## Setup

### Prerequisites

- A GitHub repository with **Issues** and **Actions** enabled
- A **GitHub Project** board configured for Kanban (columns: Todo, In Progress, In Review, Done)
- The `gh` CLI authenticated: `gh auth login` — the Agents will have exactly the same permissions as the logged-in user
- Claude Code installed: [claude.ai/code](https://claude.ai/code) or Claude Code Extension added to VS Code.

### Installing as a plugin (TBC)

**From the marketplace:**

```bash
/plugin marketplace add <repo-url>
/plugin install <agent-repo>@<agent-org>
```

**From a repository clone (based on a standard template):**

```bash
git clone <my-project-repo-url>
claude --plugin-dir /path/to/my-project-repo
```

### Verify the install

Open Claude Code in your project repo and run:

```
/spec
```

The agent will begin the spec interview. When you save, it will create a GitHub issue and return the URL. If it can't reach the `gh` CLI, check `gh auth status`.

---

## How Skills Work

Every skill follows a consistent structure:

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  Frontmatter  → name, description, trigger      │
│  Overview     → What this skill does            │
│  When to Use  → Triggering conditions           │
│  Process      → Step-by-step workflow           │
│  Rationalizations → Excuses + rebuttals         │
│  Red Flags    → Signs something's wrong         │
│  Verification → Evidence requirements           │
└─────────────────────────────────────────────────┘
```

Skills are **workflows, not reference docs**. Each has steps, checkpoints, and exit criteria. Every skill ends with evidence requirements — tests passing, build output, runtime data. "Seems right" is never sufficient.

---

## Contributing

Skills must be **specific** (actionable steps, not vague advice), **verifiable** (clear exit criteria with evidence requirements), **battle-tested** (based on real workflows), and **minimal** (only what's needed to guide the agent).

See [docs/skill-anatomy.md](docs/skill-anatomy.md) for the format specification and [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

MIT — use these skills in your projects, teams, and tools.
