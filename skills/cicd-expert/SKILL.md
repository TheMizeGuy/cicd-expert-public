---
name: cicd-expert
description: |-
  This skill should be used when the user mentions pipelines in the context of design, review, debugging, optimization, security, migration, self-hosted runners, or deployment, and the specific workflow is not obvious -- for example "design a pipeline for...", "review my CI", "pipeline is slow", "optimize this workflow", "secure my pipeline", "migrate from Jenkins to GitHub Actions", "audit my self-hosted setup". This is the main entry point for the CI/CD Expert plugin: it classifies the request, dispatches the cicd-expert agent (running on the session model), and routes to the right workflow (design / review / optimize / debug / audit-security / migrate / self-hosted-audit). If the scope is explicitly a cross-project audit (3+ repos OR "cross-project pipeline audit" OR `--team`), it routes to the `cross-project-pipeline-audit` skill instead.
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
| "cross-project", "--team", 3+ repos mentioned | `cross-project-pipeline-audit` skill |
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

If Step 1 matched a specific skill (`design-pipeline`, `review-pipeline`, `optimize-pipeline`,
`debug-pipeline`, `audit-pipeline-security`, `migrate-pipeline`, `self-hosted-audit`, or
`cross-project-pipeline-audit`), invoke that skill directly -- its own workflow-specific briefing
(findings tables, concept-mapping tables, phased plans) supersedes this generic dispatch. The snippet
below is the fallback used ONLY for the Ambiguous row, after its clarifying questions are answered and
no specific skill applies.

Dispatch the cicd-expert agent with the resolved workflow type and scope per the shared contract
`../../references/dispatch-contract.md`: `subagent_type: "cicd-expert:cicd-expert"`, model omitted
(inherits the session model; inline execution allowed per the contract's Execution mode when the
session model is already the strongest tier), description "CI/CD expert: <workflow type>", prompt
= a briefing following the contract's conventions (original user request verbatim, workflow,
working directory, scope).

## Step 4: Relay findings

Check the contract's acceptance criteria first. Present the Summary + structured findings. Offer to apply fixes one-by-one or batch-apply approved changes.

## Never do

- Dispatch `cicd-team-lead` directly by `subagent_type` -- plugin-namespaced dispatch strips the `Agent`
  tool it needs for sub-dispatch; always route cross-project scope through the
  `cross-project-pipeline-audit` skill
- Skip Step 1 classification and jump straight to a generic dispatch
- Add emojis
