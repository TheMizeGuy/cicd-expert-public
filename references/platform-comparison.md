# Platform Comparison and Tool Selection

Decision framework for choosing between hosted and self-managed CI/CD platforms.

## Overview

Platform choice is mostly a control-plane decision. Most tools can run shell commands, store artifacts, and call deployment APIs. The real differentiators are where orchestration lives, how identities and environments are modeled, what the execution fleet looks like, and how much platform engineering your team is willing to own.

## Comparison Matrix

| Platform | Control plane | Execution plane | Strengths | Tradeoffs | Best fit |
|---|---|---|---|---|---|
| GitHub Actions | GitHub-native | Hosted or self-hosted runners | Deep event integration, reusable workflows, large ecosystem, environments close to code review | Self-hosting runners does not self-host orchestration; YAML sprawl risk | Product teams standardized on GitHub |
| GitLab CI/CD | GitLab-integrated | GitLab-hosted or self-managed runners | All-in-one SCM + CI + CD + security; parent-child and component patterns; self-managed option | Broader surface means more administration if self-managed | Teams wanting one integrated platform |
| Jenkins | Jenkins controller | Jenkins agents | Maximum flexibility, 1800+ plugins, Groovy pipeline-as-code, strong enterprise adoption | Operational burden, security surface, Groovy complexity | Large orgs with unusual build/deploy needs and a platform team |
| Buildkite | Buildkite SaaS | Self-hosted agent fleets | SaaS orchestration UX with self-hosted compute, dynamic pipelines, agent queues | Less integrated for environments and approvals | SaaS orchestration + owned execution |
| Tekton/Argo | Kubernetes controllers | Kubernetes pods | Native K8s, provenance through Tekton Chains, composable with GitOps controllers | Cluster expertise required, learning curve | Kubernetes-heavy platform teams |

## Decision Heuristics By Team Shape

| Team shape | Default recommendation | Reasoning |
|---|---|---|
| Small to medium SaaS team on GitHub | GitHub Actions first | Lowest friction, strong enough for most delivery paths |
| Team standardizing on one integrated platform | GitLab CI/CD | SCM, CI, CD, environments, and security in one place |
| Large org with strong platform team and unusual needs | Jenkins or Tekton-based platform | More control, but only worth it if you can staff it |
| Company wanting SaaS orchestration, execution on owned hardware | Buildkite | Good middle ground |
| Kubernetes-heavy platform team using GitOps | Tekton for CI + Argo/Flux for CD | Native fit for cluster-centric delivery and provenance |

## Questions to Ask Before Recommending a Platform

1. Where does source code live? (GitHub, GitLab, Bitbucket, self-hosted)
2. Where does deployment happen? (Cloud, on-prem, hybrid, Kubernetes)
3. Does the team have platform engineering capacity?
4. What compliance or audit requirements exist?
5. Is there a runner fleet already in place?
6. How many repositories need CI?
7. Is the organization willing to self-manage the control plane?
8. What is the primary language/build toolchain?
9. Are there existing investments in any platform?
10. What is the deployment cadence target?
