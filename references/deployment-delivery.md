# Deployment, Environments, and Progressive Delivery

Best practices for taking validated artifacts through staging and production with rollback, approval, and observability controls.

## Core Principles

Continuous delivery is not "run deploy after tests." Mature CD systems promote a known artifact through explicit environments, use environment-specific controls to unlock production credentials, and verify rollout health with measurable criteria.

| Finding | Why it matters |
|---|---|
| Deploy immutable artifacts, then promote them | Rebuilding during promotion makes rollback and provenance weaker |
| Use environment boundaries for approvals, secrets, and change controls | GitHub environments and GitLab protected environments are designed for this |
| Progressive delivery needs both traffic control and health signals | Blue-green and canary are only safer when rollout depends on metrics |
| Rollout verification should fail closed | Fixed sleeps and advisory-only health checks create the appearance of control |
| Production deploy jobs need narrower credentials and runner trust | Deployment is where blast radius becomes operational |

## Deployment Strategy Tradeoffs

| Strategy | Strength | Weakness | Best fit |
|---|---|---|---|
| Rolling | Simple and common | Slow rollback, partial-failure exposure | Low-complexity services with good health checks |
| Blue-green | Fast rollback, clear cutover | Higher infra cost during overlap | Services where instant cutover matters |
| Canary | Safer exposure ramp, production signal | Needs traffic control and metrics | Most mature SaaS delivery paths |
| Feature flags + deploy | Decouples deploy from release | Flag debt and complexity | Teams shipping unfinished code paths |
| GitOps progressive delivery | Strong auditability, cluster reconciliation | Requires platform maturity | Kubernetes-heavy platform teams |

## Production Gate Design

| Gate | What it should prove | Default stance |
|---|---|---|
| Artifact identity | Version being promoted is the one already validated | Blocking |
| Environment approval | Right reviewer or policy approved the promotion | Blocking where required |
| Credential scope | Only deploy-time jobs access deploy credentials | Blocking |
| Readiness verification | Target is live and healthy within timeout | Blocking |
| Rollout analysis | Error rate, latency, saturation stay within policy | Blocking for canary/blue-green |
| Rollback path | Revert can be executed without manual archaeology | Blocking for production |

## GitOps

GitOps is strongest when deployment state should be cluster-reconciled and auditable. It works best when the organization wants desired state in Git and controller-driven reconciliation, not when teams just need a simple post-build deploy step.

Key tools: ArgoCD, Flux, Argo Rollouts, Flagger.

## What to Check in CD Pipelines

- Does the deployment job promote an existing artifact or silently rebuild?
- Are production secrets tied to environments rather than general repository scope?
- Is there a real approval boundary before production, or just a manual shell pause?
- Do canary or blue-green workflows include objective rollback criteria?
- Does deployment verification poll health/readiness, or does it sleep and hope?
- Are rollback instructions explicit and tested, or only implied?
- If GitOps is claimed, is desired state actually reconciled by a controller, or is the pipeline mutating live state directly?
