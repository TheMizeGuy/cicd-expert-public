# Review Checklist and Anti-Patterns

Operational review framework for creating, auditing, and improving CI/CD pipelines.

## Core Review Checklist (Seven Lenses)

| Lens | Review questions |
|---|---|
| Correctness | Does the pipeline validate the real merge path, not just a subset? Are required checks durable and meaningful? |
| Speed | Is irrelevant work skipped? Is the graph needlessly serialized? Are superseded runs canceled? |
| Security | Are permissions minimal? Are actions pinned? Are privileged runners isolated? |
| Reproducibility | Is one artifact built and promoted, with provenance or clear lineage? |
| Deployment safety | Are environments, approvals, health checks, and rollback logic explicit? |
| Operability | Are queue time, failure hotspots, and flaky checks observable? |
| Self-hosting claims | Is the team distinguishing execution, control, and deployment planes honestly? |

## High-Signal Anti-Patterns

| Anti-pattern | Why it is bad |
|---|---|
| `continue-on-error` on merge-critical validation | Converts real failures into fake-green pipelines |
| Branch protection bound to a large unstable set of jobs | Required checks become brittle and hard to evolve |
| Rebuilding artifacts during deploy | Breaks provenance and rollback confidence |
| Hard-coded sleeps in smoke or rollout verification | Wastes time and misses real readiness failures |
| Self-hosted runner pools shared across trusted and untrusted jobs | Collapses isolation boundaries |
| Full-fanout monorepo CI on every change | Burns compute while hiding the real dependency graph problem |
| Advisory-only security theater | Duplicate scans, skipped required checks, and silent bypasses create confidence theater |
| "Self-hosted" label applied to a SaaS-orchestrated system | Misstates dependencies and weakens architecture reviews |
| Unpinned third-party actions (`uses: org/action@v4`) | Tags can be moved; supply chain attack vector (CVE-2025-30066) |
| `pull_request_target` with fork code checkout | Full secrets + read-write token on attacker-controlled code |
| A new merge-gate job added without a matching branch-protection registration | Required status checks are an allowlist indexed by job id/name, not a reflection of what jobs exist -- a new job that isn't registered can run, fail, and the PR still merges. Ship the branch-protection registration (dashboard or `gh api`) in the same PR that adds the job, and have reviewers check both |

## Reference Template: Small Product Team

- Blocking PR checks: format, lint, unit tests, deterministic build, minimal security checks.
- Required merge gate: one stable aggregated status.
- Blocking protected-branch checks: integration tests and artifact creation.
- Production controls: environment approvals, scoped secrets, smoke verification, rollback note.
- Advisory or scheduled: heavier security scans, performance tests, dependency refresh reports.

## Reference Template: Monorepo Team

- Graph-aware affected-only execution.
- Stable gatekeeper job above selectively skipped leaf jobs.
- Remote caching or artifact reuse only where hit rates justify it.
- Merge queue support wired explicitly.
- Queue-time telemetry by runner class.
- Clear ownership for shared pipeline modules and cache strategy.

## Reference Template: High-Compliance Team

- Strict minimal permissions and action pinning.
- Environment-scoped approvals and secrets.
- Immutable artifact promotion with provenance retention.
- Segmented trusted deployment runners.
- Policy for blocking versus advisory scans written down.
- Rollout analysis with explicit abort criteria.
- Artifact, log, and approval retention mapped to compliance needs.

## What a CI/CD Expert Plugin Should Output

- Architecture summary: control plane, execution plane, deployment plane, and merge gate.
- Risk summary: weakest trust boundary, least reproducible step, and biggest latency sink.
- Findings ordered by severity: correctness and security first, then performance and operability.
- Concrete remediations: the smallest set of changes that materially improves trust.
- Maturity next steps: what to add later instead of overengineering now.
