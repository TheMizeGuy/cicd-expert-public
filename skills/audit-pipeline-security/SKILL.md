---
name: audit-pipeline-security
description: |-
  This skill should be used when the user asks to "audit my pipeline security", "is my CI secure", "pipeline security review", "supply chain audit", "check my workflow security", "harden my CI", "SLSA compliance", "action pinning audit", "are my actions pinned", "CI security check", or "pipeline security". Runs a CI/CD pipeline security and supply chain audit checking permissions (least privilege, GITHUB_TOKEN scoping), action pinning (SHA vs tag, CVE-2025-30066), script injection (attacker-controlled context), fork PR safety (pull_request vs pull_request_target), secret management (environment vs repo vs org scoping, OIDC), supply chain (SLSA, Sigstore, SBOMs, attestations, Scorecard, StepSecurity), runner isolation (trust zoning, ephemeral vs persistent), and workflow scanning (zizmor, CodeQL for Actions). Produces a pass/fail assessment per category with concrete remediation.
argument-hint: '[optional: scope -- all / file path / specific concern]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Audit Pipeline Security

Dispatches the cicd-expert agent with a security-workflow briefing.

## Step 1: Resolve scope

| User arg | Scope |
|---|---|
| (empty) or `all` | Full CI/CD security surface (default for security audits) |
| `<file>` | Single workflow/pipeline file |
| `<directory>` | All CI config files in dir |
| `supply-chain` | Focus on supply chain only (actions, dependencies, SBOMs) |
| `runners` | Focus on runner isolation and fleet security |

## Step 2: Dispatch cicd-expert

Dispatch per the shared contract `../../references/dispatch-contract.md`: `subagent_type:
"cicd-expert:cicd-expert"`, model omitted (inherits the session model; inline execution allowed
per the contract's Execution mode when the session model is already the strongest tier),
description "CI/CD security audit".

Prompt: read `references/security-checklist.md` (in this skill's directory) and inline its full
fenced briefing with the scope resolved from Step 1. The briefing walks seven security
categories -- permissions/least-privilege, action pinning, script injection, fork PR safety,
secret management, supply chain, runner isolation -- plus workflow scanning, each documented
Pass / Fail / N-A with evidence, then a severity-tagged findings table, supply-chain dependency
map, risk summary, verification steps, and references, with severity definitions
(CRITICAL/HIGH/MEDIUM/LOW/NIT) included.

## Step 3: Relay findings

Check the contract's acceptance criteria first. Present the security assessment summary +
critical findings. Offer to apply hardening changes.

## Never do

- Skip any security category in the checklist
- Downgrade a CRITICAL finding to avoid alarming the user
- Suggest `continue-on-error` on security checks
- Add emojis
