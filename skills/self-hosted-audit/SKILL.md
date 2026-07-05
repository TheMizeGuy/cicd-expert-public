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

Dispatch per the shared contract `../../references/dispatch-contract.md`: `subagent_type:
"cicd-expert:cicd-expert"`, model omitted (inherits the session model; inline execution allowed
per the contract's Execution mode when the session model is already the strongest tier),
description "Self-hosted CI/CD audit".

Prompt: read `references/audit-briefing.md` (in this skill's directory) and inline its full
fenced briefing with placeholders resolved from Step 1. The briefing's six deliverables, in
order: (1) three-layer assessment (execution / control / deployment planes) with an overall
verdict; (2) runner fleet assessment (ephemeral vs persistent, trust zoning, image freshness,
scaling, health monitoring, secret residue, cleanup policy); (3) supply-chain dependencies that
survive self-hosting; (4) severity-tagged findings; (5) recommendations; (6) references.

## Step 3: Relay findings

Check the contract's acceptance criteria first. Present the three-layer verdict + runner fleet assessment + key findings. Offer to apply hardening changes or update documentation to reflect accurate self-hosted claims.

## Never do

- Conflate "self-hosted runners" with "fully self-hosted pipeline"
- Skip any of the three layers in the assessment
- Assume fully self-hosted is always the goal -- hybrid is often correct and should be documented honestly
- Add emojis
