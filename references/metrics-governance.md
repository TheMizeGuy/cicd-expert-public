# Metrics, Observability, Reliability, and Governance

Operating model for measuring pipeline health, controlling risk, and proving compliance.

## DORA Metrics

DORA's five-metric model anchors delivery outcomes:

| Metric | What it measures |
|---|---|
| Deployment frequency | How often code ships to production |
| Lead time for changes | Time from commit to production |
| Change failure rate | Percentage of deploys causing failure |
| Failed deployment recovery time | Time to restore service after failure |
| Reliability | User-impactful service quality (contextual) |

## Recommended Pipeline SLI Stack

| Category | Metric | Why it matters |
|---|---|---|
| Flow | PR-to-green time | Direct developer feedback latency |
| Flow | Merge-queue wait time | Surfaces branch protection and queueing pain |
| CI runtime | Queue time by runner class | Distinguishes fleet saturation from job inefficiency |
| CI runtime | Median and p95 job duration | Shows regression and hotspot trends |
| Signal quality | Flaky retry rate | High retries mean trust is eroding |
| Signal quality | Required-check skip rate | Catches fake-green pipeline states |
| CD safety | Rollback rate and rollback time | Shows whether progressive delivery controls work |
| Outcome | DORA metrics | Connects pipeline behavior to delivery performance |

Separate waiting, running, and failing. Queue time, execution time, flaky rerun rate, and required-check pass rate diagnose different failure modes and should not be blended into one "CI is slow" metric.

## Governance Controls Worth Encoding

- Stable required checks for branch protection or rulesets.
- Workflow review ownership for security-sensitive pipeline files.
- Environment-scoped approvals and secrets.
- Artifact retention and provenance storage rules.
- Explicit policy for advisory versus blocking scans.
- Runner trust zoning and network reach documentation.
- Audit logging for who approved, deployed, and bypassed controls.

## What to Flag

- Teams tracking runtime only, with no queue-time or outcome metrics.
- Required checks that can silently skip under path filters or event mismatches.
- Production deploys with no reviewer identity or approval evidence.
- No retention strategy for build artifacts, test results, SBOMs, or provenance.
- Heavy retries masking flaky tests instead of surfacing them.
- No documented owner for pipeline configuration, self-hosted fleet ops, or deployment policy.
