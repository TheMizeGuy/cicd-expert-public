---
name: cicd-expert
description: |-
  Main entry point for the CI/CD Expert plugin -- dispatches the Opus cicd-expert agent for any CI/CD pipeline-related task. Use when the user mentions pipelines in the context of design, review, debugging, optimization, security, migration, self-hosted runners, or deployment, AND the specific workflow is not obvious. Classifies the request and routes to the right workflow (design / review / optimize / debug / audit-security / migrate / self-hosted-audit). Examples: "design a pipeline for...", "review my CI", "pipeline is slow", "optimize this workflow", "secure my pipeline", "migrate from Jenkins to GitHub Actions", "audit my self-hosted setup". If the scope is explicitly a cross-project audit (3+ repos OR "cross-project pipeline audit" OR `--team`), routes to `cross-project-pipeline-audit` skill instead.
argument-hint: '[optional: task description or scope]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# CI/CD Expert

Classifies the user's CI/CD request and dispatches the cicd-expert agent with the right workflow briefing.

## Step 1: Classify the request

| Signal | Route to |
|---|---|
| "design", "create", "build a pipeline", "set up CI" | `design-pipeline` skill |
| "review", "audit", "check", "is my pipeline good" | `review-pipeline` skill |
| "slow", "optimize", "faster", "reduce time", "cost" | `optimize-pipeline` skill |
| "failing", "broken", "debug", "error", "why is CI" | `debug-pipeline` skill |
| "security", "supply chain", "permissions", "pinning" | `audit-pipeline-security` skill |
| "migrate", "move from X to Y", "convert", "switch to" | `migrate-pipeline` skill |
| "self-hosted", "runner fleet", "ARC", "runner isolation" | `self-hosted-audit` skill |
| "cross-project", "--team", 3+ repos mentioned | Dispatch `cicd-team-lead` agent |
| Ambiguous | Ask 1-2 targeted questions, then route |

## Step 2: Resolve scope

| User arg | Scope |
|---|---|
| (empty) or `diff` | CI config files in uncommitted + staged changes |
| `staged` | Only staged CI config changes |
| `pr` | CI config diff vs main/master |
| `<file>` | Single workflow/pipeline file |
| `<directory>` | All CI config files in dir |
| `all` | Entire project CI surface |

Resolve to absolute paths. Run `git status` to confirm working tree state if scope is diff-based.

## Step 3: Dispatch

Dispatch the cicd-expert agent with the resolved workflow type and scope. Include the original user request verbatim.

```
Agent({
  description: "CI/CD expert: <workflow type>",
  subagent_type: "cicd-expert:cicd-expert",
  model: "opus",
  prompt: "<briefing with workflow, scope, working directory>"
})
```

## Step 4: Relay findings

Present the Summary + structured findings. Offer to apply fixes one-by-one or batch-apply approved changes.
