# Security Hardening

CI/CD pipeline security best practices across platforms.

## Permissions (Least Privilege)

- Deny all at workflow level (`permissions: {}`), grant per-job.
- Never give `contents: write` to PR validation jobs.
- Use `id-token: write` only on OIDC jobs.

### GitHub Actions example

```yaml
permissions: {}

jobs:
  lint:
    permissions:
      contents: read
    runs-on: ubuntu-latest
```

## OIDC for Cloud Authentication

Eliminates long-lived credentials. Short-lived tokens scoped to individual jobs.

- AWS: `aws-actions/configure-aws-credentials@v4` with `role-to-assume`
- GCP: `google-github-actions/auth@v2` with `workload_identity_provider`
- Azure: `azure/login@v2` with `client-id` + `tenant-id`

## Action Pinning

```yaml
# UNSAFE -- tag can be moved:
- uses: actions/checkout@v4

# SAFE -- immutable commit reference:
- uses: actions/checkout@a81bbbf8298c0fa03ea29cdc473d45aaaf3cb0df  # v4.2.2
```

Automate with Dependabot for `github-actions` ecosystem.

### The tj-actions Incident (CVE-2025-30066)

Supply chain attack compromised 350+ tags across `tj-actions/changed-files`, affecting 23,000+ repositories. Exfiltrated CI runner secrets via base64 in workflow logs.

Mitigations: pin all actions to SHA, rotate exposed secrets, audit for suspicious base64 strings.

## Script Injection Prevention

Attacker-controlled context properties (`body`, `title`, `head_ref`, `label`, `message`, `email`) can inject shell commands via direct interpolation.

```yaml
# VULNERABLE:
- run: echo "Title is ${{ github.event.pull_request.title }}"

# SAFE:
- env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: echo "Title is $PR_TITLE"
```

## Fork-Based PR Security

| Trigger | GITHUB_TOKEN | Secrets | Safety |
|---|---|---|---|
| `pull_request` (fork) | Read-only | None | Safe for build/test |
| `pull_request_target` | Read-write | Full | Dangerous -- never checkout fork code |
| `workflow_run` | Read-write | Full | Dangerous -- validate after completion |

## Secret Management Hierarchy

| Level | Isolation | When to use |
|---|---|---|
| Environment secrets | Strongest -- requires env ref + reviewer approval | Production deploys |
| Repository secrets | Medium -- any workflow in repo | Build/test credentials |
| Organization secrets | Weakest -- configurable repo policies | Cross-repo shared creds |
| Dependabot secrets | Isolated from Actions secrets | Private registry auth |

## Supply Chain Security

### SLSA (Supply-chain Levels for Software Artifacts)

Framework for ensuring artifact integrity. Levels 1-4 define increasing rigor:
- L1: Documentation of build process
- L2: Signed provenance from hosted build service
- L3: Hardened build platform with non-falsifiable provenance
- L4: Two-party review plus hermetic/reproducible builds

### Sigstore

Open-source signing framework. Components:
- **Cosign**: Container image and artifact signing
- **Fulcio**: Certificate authority for keyless signing
- **Rekor**: Transparency log for signatures

### SBOMs (Software Bill of Materials)

Machine-readable inventory of all components. Formats: SPDX, CycloneDX.
Generate during build, store alongside artifacts, verify during deploy.

### Scorecard

OpenSSF tool that checks repository security health: branch protection, dependency update tools, SAST, code review, pinned dependencies.

## Workflow Scanning

- **zizmor**: Open-source tool with ~24 audit rules. SARIF output.
- **CodeQL for Actions**: Scans workflow files for security vulnerabilities (GA as of April 2025).

## Runner Isolation

- Use ephemeral runners for untrusted workloads (fork PRs).
- Separate trusted deployment runners from general validation runners.
- Keep secrets out of base images; prefer short-lived credentials.
- Rebuild runner images on a schedule (not stale AMIs).
- Log runner health, queue depth, startup time, eviction rates.

## Cache and Trust Boundary Gotchas

### Fork-PR Cache Poisoning

GitHub Actions does not segregate caches by trust level. A low-privilege fork `pull_request` job can
write a cache under a predictable key; a later, more privileged workflow run that restores the same
key inherits attacker-controlled content. Never save caches from steps that touch untrusted input
(fork PR checkouts, user-supplied branch names, PR-controlled paths). CodeQL rule:
`actions-cache-poisoning-direct-cache`.

### workflow_dispatch Ref TOCTOU

A validate-ref gate on a `workflow_dispatch` input is defeated if a downstream job re-resolves
`inputs.ref` itself -- the ref can be force-moved between the job that validated it and the job that
uses it. Resolve the ref to a commit SHA once, in the validating job, and pass the SHA (not the ref
name) to every downstream job.
