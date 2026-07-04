# cicd-expert

Expert CI/CD pipeline architect, reviewer, optimizer, debugger, and security auditor for [Claude Code](https://claude.ai/claude-code).

Fable 5 agent backed by a self-contained embedded knowledge base. Covers GitHub Actions, GitLab CI/CD, Jenkins, Buildkite, Tekton, and Kubernetes-native stacks.

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

### Option 1: Clone directly

```bash
# Clone into your plugins directory
git clone https://github.com/TheMizeGuy/cicd-expert-public.git cicd-expert

# If using a directory-based plugin registry, add to your marketplace.json:
# {
#   "name": "cicd-expert",
#   "source": "./cicd-expert",
#   "version": "0.1.0"
# }
```

### Option 2: Use as a plugin directory

```bash
# Run Claude Code with the plugin
claude --plugin-dir /path/to/cicd-expert
```

### Enable in settings

Add to `~/.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "cicd-expert@your-registry": true
  }
}
```

## Agents

| Agent | Model | Purpose |
|---|---|---|
| `cicd-expert` | Fable 5 | Primary expert -- handles single-repo tasks across all workflows |
| `cicd-team-lead` | Fable 5 | Multi-repo orchestrator -- two-tier conductor pattern: dispatches Sonnet-5-xhigh recon executors per repo, dispatches Fable judgment sub-agents per scope, validates each at a gate before consolidating |

## Embedded knowledge base

The plugin carries 9 self-contained reference files in `references/`:

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

The agent reads these before every non-trivial response. No training data guessing.

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
