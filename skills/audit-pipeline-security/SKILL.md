---
name: audit-pipeline-security
description: |-
  Comprehensive CI/CD pipeline security and supply chain audit. Checks permissions (least privilege, GITHUB_TOKEN scoping), action pinning (SHA vs tag, CVE-2025-30066), script injection (attacker-controlled context), fork PR safety (pull_request vs pull_request_target), secret management (environment vs repo vs org scoping, OIDC), supply chain (SLSA, Sigstore, SBOMs, attestations, Scorecard, StepSecurity), runner isolation (trust zoning, ephemeral vs persistent), and workflow scanning (zizmor, CodeQL for Actions). Triggers on "audit my pipeline security", "is my CI secure", "pipeline security review", "supply chain audit", "check my workflow security", "harden my CI", "SLSA compliance", "action pinning audit", "are my actions pinned", "CI security check", "pipeline security". Produces pass/fail assessment per category with concrete remediation.
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

```
Agent({
  description: "CI/CD security audit",
  subagent_type: "cicd-expert:cicd-expert",
  model: "opus",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: security

SCOPE: <resolved scope>

WORKING DIRECTORY: <absolute path>

DELIVERABLES:
1. Security assessment checklist (for each, document Pass / Fail / N-A with evidence):

   PERMISSIONS AND LEAST PRIVILEGE:
   - [ ] Workflow-level `permissions: {}` (deny-all default)
   - [ ] Per-job permissions explicitly scoped
   - [ ] No `contents: write` on PR validation jobs
   - [ ] `id-token: write` only on OIDC jobs

   ACTION PINNING:
   - [ ] All third-party actions pinned to commit SHA (not tags)
   - [ ] Dependabot configured for `github-actions` ecosystem
   - [ ] No usage of compromised or abandoned actions
   - [ ] First-party actions (actions/*) at minimum pinned to major version

   SCRIPT INJECTION:
   - [ ] No direct interpolation of attacker-controlled context (body, title, head_ref, etc.)
   - [ ] All user-supplied values passed via `env:` blocks, not `${{ }}` in `run:`
   - [ ] No `eval` or dynamic code execution with user input

   FORK PR SAFETY:
   - [ ] `pull_request` (not `pull_request_target`) for fork builds
   - [ ] If `pull_request_target` is used: NO checkout of fork code
   - [ ] `workflow_run` is not used to grant fork PRs elevated permissions

   SECRET MANAGEMENT:
   - [ ] Production secrets in environment-scoped secrets (not repo-level)
   - [ ] OIDC used for cloud authentication where possible (AWS, GCP, Azure)
   - [ ] No long-lived PATs in secrets (prefer GITHUB_TOKEN or OIDC)
   - [ ] Dependabot secrets separated from Actions secrets

   SUPPLY CHAIN:
   - [ ] SBOM generation in build pipeline
   - [ ] Provenance attestation (Sigstore, Tekton Chains) where applicable
   - [ ] OpenSSF Scorecard or equivalent checks
   - [ ] StepSecurity harden-runner or egress monitoring
   - [ ] Dependency review on PRs (license, known vulnerabilities)

   RUNNER ISOLATION:
   - [ ] Ephemeral runners for untrusted workloads (fork PRs)
   - [ ] Trust zones: deployment runners separate from validation runners
   - [ ] No persistent credentials on runner disk
   - [ ] Runner images rebuilt on schedule (not stale AMIs)

   WORKFLOW SCANNING:
   - [ ] zizmor or CodeQL for Actions configured
   - [ ] SARIF results uploaded to security tab (if GitHub)

2. Severity-tagged findings table:
   | Severity | Location | Category | Finding | Evidence | Fix |
   |---|---|---|---|---|---|

3. Supply chain dependency map:
   - All remote `uses:` actions with current pinning status
   - Package registries accessed during CI
   - Container image sources
   - External URLs fetched during builds

4. Risk summary:
   - Weakest trust boundary
   - Most dangerous unpinned dependency
   - Highest-impact remediation

5. Verification steps (how to confirm fixes work)

6. References (reference files, CVEs, docs URLs)

SEVERITY DEFINITIONS:
- CRITICAL: exploitable security bug (script injection, unpinned action with known CVE, fork code with full secrets)
- HIGH: trust boundary violation or missing isolation with real attack surface
- MEDIUM: missing hardening that increases blast radius
- LOW: defense-in-depth improvement
- NIT: security hygiene preference

CONSTRAINTS:
- Walk the security checklist methodically -- do not skip categories
- For CRITICAL/HIGH: provide exact YAML remediation
- Reference CVE-2025-30066 (tj-actions) when discussing action pinning
- Check security-hardening.md for security features and mitigations
- If zizmor is available locally, run it and include results

Proceed with your standard workflow (reference files first -- especiallys 03, 09, 10, then read all CI config, then produce the audit).
```

## Step 3: Relay findings

Present the security assessment summary + critical findings. Offer to apply hardening changes.

## Never do

- Skip any security category in the checklist
- Downgrade a CRITICAL finding to avoid alarming the user
- Suggest `continue-on-error` on security checks
- Add emojis
