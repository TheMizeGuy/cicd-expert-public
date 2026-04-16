---
name: design-pipeline
description: |-
  Design a new CI/CD pipeline from scratch. Use when the user wants to create a CI/CD pipeline, set up continuous integration, build a deployment workflow, or configure CI for a new project. Covers platform selection, pipeline architecture (DAG design, merge gates, artifact strategy), security hardening, deployment path, self-hosted runner topology, and observability. Triggers on "design a pipeline", "set up CI", "create CI/CD", "build a workflow", "configure GitHub Actions", "set up GitLab CI", "create Jenkinsfile", "new pipeline for", "CI for my project". Produces complete, runnable configuration files with architecture rationale.
argument-hint: '[optional: platform preference, repo description, or constraints]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Design Pipeline

Dispatches the cicd-expert agent with a design-workflow briefing.

## Step 1: Gather context

Before dispatching, resolve:
1. **Platform**: GitHub Actions, GitLab CI, Jenkins, Buildkite, Tekton, or "recommend"
2. **Repo topology**: monorepo vs polyrepo, language(s), package manager(s)
3. **Team shape**: small product team, monorepo team, or high-compliance team (determines baseline template)
4. **Runner preference**: hosted, self-hosted, or hybrid
5. **Deployment target**: Railway, Kubernetes, VMs, PaaS, static hosting, or "none yet"
6. **Compliance needs**: any regulatory, approval, or audit requirements

If the user provided these, proceed. If not, ask the 1-2 most critical missing questions.

Read existing project files (`package.json`, `Dockerfile`, `tsconfig.json`, language markers) to infer what you can.

## Step 2: Dispatch cicd-expert

```
Agent({
  description: "Design CI/CD pipeline",
  subagent_type: "cicd-expert:cicd-expert",
  model: "opus",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: design

CONTEXT:
- Platform: <resolved>
- Repo topology: <resolved>
- Team shape: <small-product / monorepo / high-compliance>
- Runner preference: <hosted / self-hosted / hybrid>
- Deployment target: <resolved>
- Compliance needs: <resolved>
- Working directory: <absolute path>

DELIVERABLES:
1. Architecture summary:
   - Pipeline DAG diagram (text)
   - Control plane, execution plane, deployment plane
   - Merge gate strategy
   - Artifact promotion path

2. Complete configuration files:
   - All workflow/pipeline YAML files
   - Branch protection / ruleset recommendations
   - Environment definitions with approval rules
   - Secret scoping recommendations

3. Security baseline:
   - Permissions blocks (least privilege)
   - Action pinning strategy
   - OIDC configuration (if applicable)
   - Supply chain controls (Dependabot, Scorecard)

4. Optimization built-in:
   - Path filtering for affected-only execution
   - Concurrency controls for duplicate cancellation
   - Cache strategy (keys, paths, expected hit rate)
   - Job parallelism vs serialization rationale

5. Observability hooks:
   - What to measure (queue time, execution time, flaky rate)
   - Where to start with DORA metrics

6. Maturity roadmap:
   - What to add later vs what to ship now
   - Progressive delivery readiness

TEMPLATE SELECTION:
Use the reference templates from review-checklist.md as the starting baseline:
- Small product team: blocking PR checks + stable merge gate + environment approvals
- Monorepo team: graph-aware affected-only + stable gatekeeper + remote caching
- High-compliance team: strict permissions + pinning + immutable artifacts + provenance + rollout analysis

Proceed with your standard workflow (reference files first, then codebase inspection, then produce the design).
```

## Step 3: Relay and apply

Present the architecture summary + key files. Offer to write the configuration files to the project directory.

## Never do

- Design a pipeline without reading pipeline-architecture.md and review-checklist.md
- Recommend a platform without reading platform-comparison.md
- Produce a design without addressing all seven review lenses
- Add emojis
