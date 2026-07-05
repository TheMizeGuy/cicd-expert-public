# cicd-expert

Expert CI/CD pipeline architect, reviewer, optimizer, debugger, and security auditor for [Claude Code](https://claude.ai/claude-code).

Runs on the session model -- always the strongest available Claude -- and is backed by a self-contained embedded knowledge base. Covers GitHub Actions, GitLab CI/CD, Jenkins, Buildkite, Tekton, and Kubernetes-native stacks.

## What it does

| Skill | Slash command | What it does |
|---|---|---|
| `design-pipeline` | `/cicd-expert:design-pipeline` | Design new pipelines from scratch with architecture rationale |
| `review-pipeline` | `/cicd-expert:review-pipeline [scope]` | Seven-lens review (correctness, speed, security, reproducibility, deployment, operability, self-hosting) |
| `optimize-pipeline` | `/cicd-expert:optimize-pipeline` | Systematic optimization using the hierarchy (skip > shorten > cache > fleet > toolchain) |
| `debug-pipeline` | `/cicd-expert:debug-pipeline [error]` | Systematic debugging (reproduce, isolate, hypothesize, test, confirm, fix, verify) |
| `audit-pipeline-security` | `/cicd-expert:audit-pipeline-security` | 9-category security checklist: permissions, pinning, injection, fork safety, OIDC, supply chain, runners |
| `migrate-pipeline` | `/cicd-expert:migrate-pipeline [src -> tgt]` | Cross-platform migration with concept mapping, phased plan, rollback per stage |
| `self-hosted-audit` | `/cicd-expert:self-hosted-audit` | Three-layer model assessment (execution/control/deployment planes) |
| `cross-project-pipeline-audit` | `/cicd-expert:cross-project-pipeline-audit --team` | Multi-repo team-mode audit with parallel agents |
| `cicd-expert` | `/cicd-expert [description]` | Natural language router -- classifies your request and dispatches the right workflow |

## Key frameworks

### Seven-lens review

Every review evaluates pipelines across all seven dimensions:

1. **Correctness** -- validates the real merge path
2. **Speed** -- skips irrelevant work, minimal serialization
3. **Security** -- least privilege, pinned actions, isolated runners
4. **Reproducibility** -- one artifact built and promoted with provenance
5. **Deployment safety** -- environments, approvals, health checks, rollback
6. **Operability** -- queue time, failure hotspots, flaky checks observable
7. **Self-hosting accuracy** -- three-layer model applied honestly

### Three-layer self-hosted model

"Self-hosted runners" is not the same as "fully self-hosted pipeline."

| Layer | Question |
|---|---|
| Execution plane | Where do jobs run? |
| Control plane | Who schedules, authorizes, stores state? |
| Deployment plane | Where do artifacts get promoted and run? |

### Optimization hierarchy

Applied in order -- never jump to toolchain tuning first:

1. Skip irrelevant work (path filtering, affected-only, cancel superseded)
2. Shorten the critical path (remove unnecessary `needs:` edges)
3. Cache smarter (dependency caches, Docker layers, artifact reuse)
4. Fix fleet topology (scale runners, move lightweight jobs off scarce machines)
5. Optimize the toolchain (package managers, compiler caches, matrix)

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/TheMizeGuy/cicd-expert-public.git

# 2. Install the plugin
claude plugin install cicd-expert@cicd-expert-public

# 3. Restart Claude Code for the plugin to load
```

After restart, verify with `claude plugin list`. Updates ship through the same channel: when a new release lands, run `claude plugin marketplace update cicd-expert-public` then `claude plugin update cicd-expert@cicd-expert-public`, or accept the update prompt in `/plugin`.

Manual alternative: `git clone https://github.com/TheMizeGuy/cicd-expert-public.git` and load with `claude --plugin-dir <path>`.

## Agents

| Agent | Model | Purpose |
|---|---|---|
| `cicd-expert` | Session model (always the strongest available Claude) | Primary expert -- handles single-repo tasks across all workflows |
| `cicd-team-lead` | Session model (always the strongest available Claude) | Multi-repo orchestrator -- two-tier conductor pattern: dispatches conductor-selected recon executors (Sonnet 5 @ `xhigh` or Opus 4.8) per repo, dispatches session-model judgment sub-agents per scope, validates each at a gate before consolidating |

## Embedded knowledge base

The plugin carries 9 self-contained domain reference files in `references/`, read by the
`cicd-expert` agent before every non-trivial response (no training data guessing):

| File | Coverage |
|---|---|
| `pipeline-architecture.md` | DAG design, merge gates, artifact promotion, quality gate stack |
| `security-hardening.md` | Permissions, OIDC, SHA pinning, script injection, fork safety, SLSA, Sigstore, SBOMs |
| `optimization-patterns.md` | The optimization hierarchy, caching, monorepo rules, cost control |
| `self-hosted-runners.md` | Three-layer model, runner fleet design, audit checklist |
| `deployment-delivery.md` | Environments, progressive delivery, blue-green/canary, GitOps, rollback |
| `metrics-governance.md` | DORA metrics, pipeline SLIs, governance controls |
| `platform-comparison.md` | GitHub Actions vs GitLab vs Jenkins vs Buildkite vs Tekton |
| `review-checklist.md` | Seven-lens framework, anti-patterns, reference templates |
| `github-actions-gotchas.md` | Merge queue gotchas, local composite action ordering and lifecycle-hook limits |

A tenth file, `references/dispatch-contract.md`, is not domain knowledge -- it is the shared
dispatch mechanics (Agent call template, briefing conventions, execution mode, acceptance
criteria) that the seven single-workflow skills point to instead of each repeating the same
boilerplate. Two skills also carry their own long checklist briefings in a skill-local
`references/` subdirectory (`skills/audit-pipeline-security/references/security-checklist.md`,
`skills/self-hosted-audit/references/audit-briefing.md`) for the same reason.

## Documentation

- [`USAGE.md`](USAGE.md) -- quickstart, one worked walkthrough per skill, troubleshooting table.
- [`CHANGELOG.md`](CHANGELOG.md) -- version history.

## Anti-patterns detected

The plugin flags these high-signal anti-patterns automatically:

- `continue-on-error` on merge-critical validation
- Branch protection bound to an unstable set of jobs
- Rebuilding artifacts during deploy
- Hard-coded sleeps in smoke or rollout verification
- Shared runner pools across trust zones
- Full-fanout monorepo CI on every change
- Advisory-only security theater
- Mislabeled self-hosted claims
- Unpinned third-party actions (CVE-2025-30066)
- `pull_request_target` with fork code checkout

## Platforms covered

GitHub Actions, GitLab CI/CD, Jenkins, Buildkite, Tekton, Woodpecker, Drone, ArgoCD, Flux, Flagger.

## Optional dependencies

- **context7 plugin** (recommended): For live CI platform documentation lookups. Install from the Claude Code plugin marketplace.

## License

MIT
