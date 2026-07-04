# Self-Hosted CI/CD and Runner Fleet Design

Design patterns, tradeoffs, and audit criteria for self-hosted runners and fully self-managed pipeline stacks.

## The Three-Layer Model

"Self-hosted CI/CD" is often used imprecisely. Reason about it as three separate layers:

| Layer | Question | Typical examples |
|---|---|---|
| Execution plane | Where do jobs actually run? | GitHub self-hosted runners, GitLab runners, Jenkins agents, Buildkite agents |
| Control plane | Who schedules, authorizes, stores workflow state, and emits events? | GitHub Actions, GitLab, Jenkins controller, Buildkite, Tekton controllers |
| Deployment plane | Where do artifacts get promoted and run? | Kubernetes clusters, VMs, PaaS targets, internal registries, GitOps controllers |

Moving `runs-on` to self-hosted only changes the execution plane. A truly self-managed pipeline story requires reviewing all three layers. That distinction matters for reliability, security, and compliance.

## Key Findings

| Finding | Why it matters |
|---|---|
| Prefer ephemeral runners for mixed-trust or untrusted workloads | Reduces persistence of workspace state, secrets residue, and cross-job contamination |
| Segment runner pools by trust zone and capability | Deployment jobs, fork PR validation, privileged image builds, and general unit tests should not share workers |
| Persistent disks are an optimization choice, not a safety default | They can improve cache hit rate but make stale workspaces and secret residue easier to carry across runs |
| Self-hosted fleets still need orchestration discipline | ARC, GitLab runners, Jenkins agents, and Buildkite agent queues all require image updates, scaling policy, and health monitoring |
| Scarce heavyweight runners should execute heavyweight work | Path filtering, report aggregation, bot comments, and advisory scans belong on cheaper or hosted workers |
| Privileged executors deserve special scrutiny | Shell executors, privileged containers, or broad host mounts collapse isolation between jobs |
| SaaS control planes with self-hosted workers are hybrid architectures | That is often correct, but teams should describe it accurately when auditing dependencies |

## Platform-Specific Notes

| Platform | Operational note |
|---|---|
| GitHub Actions | Self-hosted runners and ARC are execution-plane tools; use ephemeral runners where possible |
| GitLab | Runner executor choice is a security decision; shell and privileged container models carry different risk |
| Jenkins | Once the controller is compromised, agents are generally within blast radius |
| Buildkite | Queue design, token management, and host hardening determine most of the real security posture |
| Tekton | Self-hosting is controller- and cluster-centric; provenance through Tekton Chains |

## Runner Fleet Best Practices

- Use immutable runner images and rebuild them on a schedule.
- Keep secrets out of base images; prefer short-lived credentials.
- Separate trusted deployment runners from general validation runners.
- Log runner health, queue depth, startup time, eviction/retry rates, and failed-job causes.
- Make cleanup explicit for any non-ephemeral worker model.
- Treat third-party marketplace actions, package registries, and browser download CDNs as part of the self-hosted supply chain.

## Managed Ephemeral Runner Services

A third runner category sits between fully hosted and fully self-hosted: managed ephemeral runner
services (for example, Namespace.so) that provision short-lived, vendor-operated VMs on demand. Frame
this as a runner-type category to identify, not a specific vendor recommendation:

- Labels select machine shape (CPU/RAM, and OS/arch); right-sizing the shape to the job matters as
  much as it does for self-managed fleets.
- Cache mounts typically require an additional cache-enabled label variant on the `runs-on` value --
  a plain compute label does not automatically get a cache volume.
- Cache volumes may enforce per-branch write allow-lists that silently discard writes from
  non-allowed branches; verify actual cache hit rates rather than assuming a cache step is doing
  anything.
- A job stuck queued indefinitely ("waiting for a runner") usually means the vendor's GitHub App (or
  equivalent integration) lacks access to that repository, not a capacity problem.

These are vendor-class characteristics of the managed-ephemeral-runner model, not an endorsement of
any specific provider -- evaluate against the same three-layer model and audit checklist as any other
execution plane.

## Self-Hosted Audit Checklist

1. **Execution plane**: Which jobs still run on hosted runners?
2. **Control plane**: Are schedules, artifacts, branch protection, Dependabot, and PR events still required from the hosted platform?
3. **Deploy target**: Is deploy coupled to a hosted PaaS via hard-coded URLs or provider manifests?
4. **Dependency supply chain**: npm/PyPI/apt/Docker Hub/Playwright browser CDN/public git clones.
5. **Marketplace action supply chain**: Remote `uses:` actions are external dependencies even on self-hosted runners.
6. **Verification tooling**: Helper scripts often hard-code production SaaS URLs long after the main code is supposedly portable.

Moving `runs-on` to self-hosted is only the first step. Fully self-hosted delivery usually also requires vendored actions, mirrored registries, internal artifact storage, and a deploy path that pushes directly to owned infrastructure.
