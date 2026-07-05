---
name: cross-project-pipeline-audit
description: |-
  This skill should be used when the user asks to run a "cross-project pipeline audit", "multi-repo pipeline audit", or "audit all my pipelines" across 3+ repositories in parallel, or passes the `--team` flag on other skills. Dispatches the cicd-team-lead agent (running on the session model), which maps each repo's pipeline surface, partitions work into focused scopes (build validation, security posture, deployment path, runner fleet, observability, cross-repo patterns), dispatches parallel cicd-expert sub-agents (bounded by the fan-out budget), and consolidates findings into a unified report with overall verdict + per-repo deltas + cross-cutting pattern detection. Do NOT trigger on generic "audit my pipeline" (single-repo -- use `review-pipeline` or `audit-pipeline-security`).
argument-hint: '[optional: --repos=<repo1,repo2,...> or repo paths]'
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

Read `${CLAUDE_PLUGIN_ROOT}/agents/cicd-team-lead.md` and inline its body (everything below the
frontmatter) as the prompt prefix. Do NOT dispatch via `subagent_type: "cicd-expert:cicd-team-lead"` --
plugin-namespaced dispatch silently strips the `Agent` tool the team lead needs for sub-dispatch (see the
agent file's RUNTIME DISPATCH NOTE -- a known Claude Code platform limitation).

```
Agent({
  description: "Cross-project CI/CD audit: N repos",
  subagent_type: "general-purpose",
  // omit model -- inherits the session model (always the strongest available Claude)
  prompt: "<cicd-team-lead.md body>\n\n<briefing with repo list, platforms detected, and deliverable format>"
})
```

The team lead runs a two-tier conductor-executor dispatch:
1. Map each repo's pipeline surface
2. Partition into 4-10 focused scopes
3. Dispatch executor-class recon (`Agent({subagent_type: "general-purpose", model: "sonnet"})` at
   xhigh effort) per repo or scope -- pipeline-surface mapping, workflow/job/trigger counts, runner-label
   inventories (identification by type, not migration doctrine), per-file mechanical transforms. Each
   recon dispatch carries a SHARED CONTEXT pointer, non-overlapping repo/file ownership, an ESCALATE
   rule, and a BLACKBOARD path. These recon executors are exempt from the ≤10/wave, ≤20/turn fan-out cap
   (conductor-selected executors -- Sonnet 5 @ `xhigh` or Opus 4.8 -- scale to natural breadth) but still need a hard per-repo iteration cap.
4. Validate each executor's blackboard before accepting its findings (read it, spot-check against the
   repo, one re-dispatch max on failure)
5. Synthesize verdicts, cross-repo pattern detection, and the consolidated report on the session model
   (always the strongest available Claude) -- judgment and synthesis are never delegated to any
   executor. This conductor-class stage IS bound by the ≤10/wave, ≤20/turn session-model cap.

Ground the consolidated report format and cross-cutting anti-pattern detection in `review-checklist.md`
("What a CI/CD Expert Plugin Should Output": architecture summary, risk summary, severity-ordered
findings, concrete remediations, maturity next steps; High-Signal Anti-Patterns table).

## Step 3: Relay findings

Present the unified cross-project report with overall verdict, per-repo summary, cross-cutting patterns, and recommended action plan.

## Never do

- Dispatch for a single repo (use `review-pipeline` instead)
- Exceed the fan-out budget (≤10 agents/wave, ≤20/turn) for session-model dispatches without
  explicit user sign-off -- conductor-selected recon executors are exempt from this cap but still need a
  hard iteration cap (>20 Opus executors in one turn still needs sign-off)
- Skip the validation gate -- accept an executor's recon findings without reading its blackboard and
  spot-checking against the repo
- Produce separate reports per repo instead of one unified report
- Add emojis
