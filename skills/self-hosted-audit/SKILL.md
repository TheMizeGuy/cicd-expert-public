---
name: self-hosted-audit
description: |-
  This skill should be used when the user asks to "audit my self-hosted CI", "is my pipeline really self-hosted", "review my runner fleet", "self-hosted runner audit", "ARC review", "runner isolation check", "are my runners configured correctly", "self-hosted security", "runner fleet design review", or "audit self-hosted claims". Audits self-hosted CI/CD claims and runner fleet design, applying the three-layer model (execution plane, control plane, deployment plane) to evaluate whether a pipeline is truly self-hosted, hybrid, or mislabeled. Checks runner fleet practices -- ephemeral vs persistent, trust zoning, image freshness, scaling strategy, health monitoring, secret residue -- and supply chain dependencies that survive the self-hosted migration (marketplace actions, package registries, browser CDNs). Produces a layer-by-layer assessment with concrete findings.
argument-hint: '[optional: scope -- all / runner config files / specific concern]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Self-Hosted Audit

Dispatches the cicd-expert agent with a self-hosted-workflow briefing.

## Step 1: Gather context

Read CI config files and identify:
1. Which jobs use `runs-on: self-hosted` vs hosted runners
2. Runner labels, groups, and fleet configuration (ARC, GitLab runner config, Jenkins agent config)
3. Deployment targets (Railway, Kubernetes, VMs, PaaS)
4. External dependencies (marketplace actions, registries, CDNs)

## Step 2: Dispatch cicd-expert

```
Agent({
  description: "Self-hosted CI/CD audit",
  subagent_type: "cicd-expert:cicd-expert",
  model: "fable",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: self-hosted

WORKING DIRECTORY: <absolute path>
CI CONFIG FILES: <list of files found>
RUNNER CONFIG FILES: <if found: ARC values.yaml, .gitlab-ci.yml runner tags, Jenkinsfile agent blocks>

DELIVERABLES:
1. Three-layer assessment:

   EXECUTION PLANE (where do jobs run?):
   | Job/workflow | Runner type | Label | Self-hosted? | Notes |
   |---|---|---|---|---|
   For each job: which runner, is it truly self-hosted, any jobs still on hosted runners?

   CONTROL PLANE (who orchestrates?):
   - Scheduling: <GitHub Actions / GitLab / Jenkins controller / Buildkite / Tekton>
   - Artifact storage: <GitHub / self-hosted / S3>
   - Branch protection/merge policy: <GitHub / GitLab / self-hosted>
   - Event system: <GitHub webhooks / self-hosted>
   - Self-hosted? <yes/no/hybrid -- with evidence>

   DEPLOYMENT PLANE (where do artifacts run?):
   - Deploy targets: <Railway / K8s / VMs / etc.>
   - Deploy mechanism: <direct push / GitOps / provider API>
   - Self-hosted? <yes/no/hybrid -- with evidence>

   OVERALL VERDICT:
   - [ ] Fully self-hosted (all three layers under own control)
   - [ ] Hybrid (execution self-hosted, control/deploy external)
   - [ ] Execution-only self-hosted (most common mislabeling)
   - [ ] Not meaningfully self-hosted

2. Runner fleet assessment:
   - Ephemeral vs persistent: <which model, risk of stale state>
   - Trust zoning: <are deploy runners separated from validation runners?>
   - Image freshness: <when was the runner image last rebuilt?>
   - Scaling strategy: <static / ARC / GitLab autoscaling / manual>
   - Health monitoring: <queue depth, startup time, eviction rate>
   - Secret residue: <could secrets persist on runner disk between jobs?>
   - Cleanup policy: <explicit cleanup steps or relying on ephemeral model?>

3. Supply chain dependencies that survive the self-hosted migration:
   - Remote `uses:` actions (marketplace actions are external even on self-hosted runners)
   - Package registries (npm/PyPI/apt/Docker Hub)
   - Browser CDNs (Playwright browser downloads)
   - External git clones
   - Container base images from public registries
   For each: is it vendored/mirrored, or still an external dependency?

4. Severity-tagged findings:
   | Severity | Layer | Finding | Evidence | Fix |
   |---|---|---|---|---|

5. Recommendations:
   - If truly aiming for fully self-hosted: what remains to migrate
   - If hybrid is acceptable: how to document it honestly
   - Runner fleet best practices to adopt

6. References (reference files, docs URLs)

CONSTRAINTS:
- Apply the three-layer model rigorously -- do not conflate layers
- Read self-hosted-runners.md as primary reference
- Check for common mislabeling: "self-hosted" when only execution plane changed
- Verify scripts and helper tools for hardcoded external URLs
- Flag persistent runners with shared state as a security risk
- Do NOT recommend fully self-hosted as always better -- hybrid is often correct

Proceed with your standard workflow (reference files first -- especially 15 for self-hosted learnings, then read all CI and runner config, then produce the audit).
```

## Step 3: Relay findings

Present the three-layer verdict + runner fleet assessment + key findings. Offer to apply hardening changes or update documentation to reflect accurate self-hosted claims.

## Never do

- Conflate "self-hosted runners" with "fully self-hosted pipeline"
- Skip any of the three layers in the assessment
- Assume fully self-hosted is always the goal -- hybrid is often correct and should be documented honestly
- Add emojis
