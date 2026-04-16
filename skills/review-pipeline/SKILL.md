---
name: review-pipeline
description: |-
  Comprehensive CI/CD pipeline review -- workflow files, security posture, performance, deployment path, and self-hosted claims. Checks across all seven lenses: correctness, speed, security, reproducibility, deployment safety, operability, and self-hosting accuracy. Triggers on "review my pipeline", "review my CI", "audit my workflows", "check my CI/CD", "is my pipeline good", "review this workflow file", "pipeline code review", "CI review", "feedback on my pipeline", "review my GitHub Actions", "audit my GitLab CI", "check my Jenkinsfile". Produces severity-tagged findings (CRITICAL/HIGH/MEDIUM/LOW/NIT) with concrete fixes and evidence-backed citations. For security-only audits, use audit-pipeline-security; for performance-only, use optimize-pipeline.
argument-hint: '[optional: scope -- diff / staged / pr / all / file path]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Review Pipeline

Dispatches the cicd-expert agent with a review-workflow briefing.

## Step 1: Resolve scope

| User arg | Scope |
|---|---|
| (empty) or `diff` | Uncommitted + staged CI config changes (default) |
| `staged` | Only staged changes |
| `pr` | Diff vs main/master |
| `<file>` | Single workflow/pipeline file |
| `<directory>` | All CI config files in dir (e.g., `.github/workflows/`) |
| `all` | Entire project CI surface |

Resolve to absolute paths. For diff-based scopes, run `git diff --name-only` to find CI-relevant files.

## Step 2: Dispatch cicd-expert

```
Agent({
  description: "Pipeline review: <scope>",
  subagent_type: "cicd-expert:cicd-expert",
  model: "opus",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: review

SCOPE: <resolved scope -- files, diff range>

WORKING DIRECTORY: <absolute path>

DELIVERABLES:
1. File inventory (what you read, platform detected)

2. Severity-tagged findings table:

   | Severity | Location | Lens | Finding | Evidence | Fix |
   |---|---|---|---|---|---|
   | CRITICAL | .github/workflows/ci.yml:15 | Security | Unpinned third-party action | 03 [P] | <yaml block> |
   | HIGH | ... | Speed | Unnecessary serialization | 14 [PxN] | ... |
   | ... | ... | ... | ... | ... | ... |

3. Seven-lens coverage checklist (explicitly note what was checked):
   - [pass/fail/n-a] Correctness: validates real merge path, required checks are durable
   - [pass/fail/n-a] Speed: irrelevant work skipped, DAG not needlessly serialized, superseded runs canceled
   - [pass/fail/n-a] Security (check each sub-item):
     - Permissions: workflow-level deny-all, per-job least privilege
     - Action pinning: all third-party actions pinned to SHA
     - Script injection: no attacker-controlled context in `run:` blocks
     - Fork PR safety: `pull_request` not `pull_request_target` for fork builds
     - Secret scoping: production secrets in environment-scoped, not repo-level
     - OIDC: short-lived credentials for cloud auth where possible
     - Supply chain: SBOM, provenance, dependency review
     - Runner isolation: trust zones, ephemeral for untrusted workloads
     - Workflow scanning: zizmor or CodeQL for Actions
   - [pass/fail/n-a] Reproducibility: one artifact built and promoted, provenance or lineage present
   - [pass/fail/n-a] Deployment safety: environments with approvals, health checks, rollback explicit
   - [pass/fail/n-a] Operability: queue time/failure hotspots observable, flaky checks surfaced
   - [pass/fail/n-a] Self-hosting accuracy: execution/control/deployment planes distinguished honestly

4. Anti-pattern scan (check for all high-signal anti-patterns from 18 - Review Checklist):
   - continue-on-error on merge-critical checks
   - Brittle required check lists
   - Artifact rebuilding during deploy
   - Hard-coded sleeps
   - Shared runner pools across trust zones
   - Full-fanout monorepo CI
   - Advisory-only security theater
   - Mislabeled self-hosted claims

5. Top 3 priorities (impact x effort)
6. Verification steps the user can run after applying fixes
7. References (reference files, docs URLs)

SEVERITY DEFINITIONS:
- CRITICAL: security or correctness bug; ship-blocker
- HIGH: performance or trust boundary risk with real impact
- MEDIUM: pattern deviation, technical debt, anti-pattern
- LOW: style, clarity, convention
- NIT: preference / taste

CONSTRAINTS:
- For CRITICAL/HIGH findings, provide runnable YAML fixes
- Preserve behavioral contracts -- never suggest changing trigger events, required check names, or environment names without rationale
- If the pipeline uses reusable workflows or composite actions, review those too

Proceed with your standard workflow (reference files first, then read all CI config files, then produce the review).
```

## Step 3: Relay findings

Present the Summary + severity-tagged table. Offer to apply fixes one-by-one starting with CRITICAL, or to dispatch a fresh cicd-expert agent in "fix" mode to batch-apply approved findings.

## Never do

- Produce a review without checking all seven lenses
- Collapse severity -- CRITICAL stays CRITICAL
- Produce a review without at least one evidence-backed `[P]` or `[PxN]` finding
- Suggest changing trigger events, required check names, or environment names without calling out downstream branch protection rules and status check consumers
- Add emojis
