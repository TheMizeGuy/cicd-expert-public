---
name: cross-project-pipeline-audit
description: |-
  Run a comprehensive CI/CD pipeline audit across 3+ repositories in parallel. Dispatches the cicd-team-lead Opus agent, which maps each repo's pipeline surface, partitions work into focused scopes (build validation, security posture, deployment path, runner fleet, observability, cross-repo patterns), dispatches parallel cicd-expert sub-agents (max 4 concurrent), and consolidates findings into a unified report with overall verdict + per-repo deltas + cross-cutting pattern detection. Triggers ONLY on explicit phrases -- "cross-project pipeline audit", "multi-repo pipeline audit", "audit all my pipelines", OR the `--team` flag on other skills. Do NOT trigger on generic "audit my pipeline" (single-repo -- use `review-pipeline` or `audit-pipeline-security`).
argument-hint: '[--repos=<repo1,repo2,...> or repo paths]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Cross-Project Pipeline Audit

Dispatches the cicd-team-lead agent for multi-repo pipeline orchestration.

## Step 1: Resolve repos

Parse user input for repo paths:
- `--repos=api,web,mobile` -- resolve to absolute paths
- Explicit paths provided by user
- If no repos specified, scan parent directory for repos with CI config

Confirm repo list and CI platform for each before dispatching.

## Step 2: Dispatch cicd-team-lead

```
Agent({
  description: "Cross-project CI/CD audit: N repos",
  subagent_type: "cicd-expert:cicd-team-lead",
  model: "opus",
  prompt: "<briefing with repo list, platforms detected, and deliverable format>"
})
```

The team lead will:
1. Map each repo's pipeline surface
2. Partition into 4-10 focused scopes
3. Dispatch parallel cicd-expert agents (max 4)
4. Consolidate into unified report

## Step 3: Relay findings

Present the unified cross-project report with overall verdict, per-repo summary, cross-cutting patterns, and recommended action plan.

## Never do

- Dispatch for a single repo (use `review-pipeline` instead)
- Dispatch more than 4 concurrent sub-agents
- Produce separate reports per repo instead of one unified report
- Add emojis
