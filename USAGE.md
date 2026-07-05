# cicd-expert -- Usage Guide

The complete guide to driving `cicd-expert`: what each skill does, how to invoke it, what you get back, and how to troubleshoot it. For the framework and knowledge-base details, see [`README.md`](README.md).

## Quickstart

1. Install (see [Installation](README.md#installation) in the README).
2. Describe your task in natural language -- you do not need to name a skill:
   ```
   /cicd-expert "review my GitHub Actions workflows"
   ```
3. The `cicd-expert` router skill classifies the request and dispatches the `cicd-expert` agent (or a named skill, if the router matches one directly) with a workflow-specific briefing.
4. The agent reads the relevant `references/*.md` files first, then your CI configuration, then produces a report following a fixed structure: Summary, Context, Findings, Configuration changes, Verification steps, References.
5. Review the findings. For CRITICAL/HIGH items the agent proposes a runnable fix (YAML or config diff) -- approve it before it is applied.

Every non-obvious claim in a report carries a confidence grade (`[P]` primary source, `[S]` secondary, `[PxN]` pattern seen N times, `[V]` tool-verified in your repo, `[recall]` unverified -- treat `[recall]` items as claims to double-check).

## One walkthrough per skill

### `design-pipeline` -- new pipeline from scratch

```
/cicd-expert:design-pipeline "GitHub Actions for a Node/TypeScript monorepo with three deployable services"
```

**What happens:** the skill reads your `package.json`, lockfile, `Dockerfile`, and language config to infer the toolchain, then dispatches `cicd-expert` with a design briefing. The agent reads `pipeline-architecture.md`, `platform-comparison.md`, and `security-hardening.md`, then produces complete workflow files (not pseudocode) plus an architecture rationale: DAG shape, merge-gate placement, artifact promotion strategy, and the security defaults it applied.

**Output shape:** one or more ready-to-commit CI config files, a short rationale per major decision, and a verification checklist (which checks must pass before the pipeline is considered working).

### `review-pipeline` -- seven-lens review

```
/cicd-expert:review-pipeline diff
```

**What happens:** the skill resolves scope (`diff` = your uncommitted + staged changes; also accepts `staged`, `pr`, `all`, or a file path), reads the matching CI config, and dispatches `cicd-expert` with a review briefing covering correctness, speed, security, reproducibility, deployment safety, operability, and self-hosting accuracy.

**Output shape:** a severity-tagged findings table (`CRITICAL`/`HIGH`/`MEDIUM`/`LOW`/`NIT`), each row with file:line, the violated lens, graded evidence, and a runnable fix.

### `optimize-pipeline` -- speed and cost

```
/cicd-expert:optimize-pipeline "our CI takes 45 minutes and it's mostly queue time"
```

**What happens:** the agent first measures whether the bottleneck is queue time, execution time, unnecessary serialization, or fleet capacity (never guesses), then applies the optimization hierarchy top-down: skip irrelevant work, shorten the critical path, cache smarter, fix fleet topology, tune the toolchain last.

**Output shape:** a bottleneck diagnosis, a before/after configuration diff per change, and an expected-impact estimate for each.

### `debug-pipeline` -- systematic failure diagnosis

```
/cicd-expert:debug-pipeline "deploy job hangs after the build step"
```

**What happens:** the skill gathers the failing workflow file and recent CI-config git history first. The agent then works the systematic loop -- reproduce, isolate, hypothesize, test the hypothesis, confirm root cause, fix, verify -- and never proposes a fix before confirming the cause.

**Output shape:** root cause analysis (with the confirming evidence) and a concrete fix, offered for direct application.

### `audit-pipeline-security` -- security and supply chain

```
/cicd-expert:audit-pipeline-security supply-chain
```

**What happens:** resolves scope (`all` by default; also accepts a file, a directory, `supply-chain`, or `runners`), then walks the full security-checklist briefing in `skills/audit-pipeline-security/references/security-checklist.md` covering permissions, action pinning (SHA vs tag, including CVE-2025-30066), script injection, fork PR safety (`pull_request` vs `pull_request_target`), secret management, supply chain (SLSA, Sigstore, SBOMs, Scorecard), and runner isolation.

**Output shape:** a Pass/Fail/N-A table per category with evidence, a severity-tagged findings table, and a supply-chain dependency map.

### `migrate-pipeline` -- cross-platform migration

```
/cicd-expert:migrate-pipeline "Jenkins to GitHub Actions"
```

**What happens:** if source and target are not both stated, the skill asks first. The agent reads all source pipeline files, builds a concept mapping (agents/stages -> jobs/steps, credentials -> secrets, plugins -> actions/equivalent tooling), and produces a phased plan with a rollback point at every stage.

**Output shape:** a concept-mapping table, a phased migration plan, and target-platform configuration files for Phase 1 (later phases produced on request).

### `self-hosted-audit` -- three-layer self-hosted assessment

```
/cicd-expert:self-hosted-audit
```

**What happens:** the agent classifies every job into the execution / control / deployment planes (see [README: Three-layer self-hosted model](README.md#three-layer-self-hosted-model)), assesses runner fleet practices (ephemeral vs persistent, trust zoning, image freshness, secret residue, cleanup policy) when self-hosted runners are present, and checks whether marketplace actions, package registries, and CDNs still leave supply-chain dependencies after migrating execution off hosted runners.

**Output shape:** a three-layer verdict (fully self-hosted / hybrid / execution-only / not meaningfully self-hosted), a runner fleet assessment, and severity-tagged findings.

### `cross-project-pipeline-audit` -- multi-repo team mode

```
/cicd-expert:cross-project-pipeline-audit --repos=api,web,worker
```

**What happens:** the `cicd-team-lead` agent maps each repo's pipeline surface, partitions the work into 4-10 focused scopes (build validation, security posture, deployment path, runner fleet, observability, cross-repo patterns), and runs a two-tier dispatch -- conductor-selected executors (Sonnet 5 @ `xhigh`, or Opus 4.8 when the recon needs deeper judgment) do per-repo recon, session-model sub-agents render judgment per scope, each validated at a gate before the team lead synthesizes the consolidated report. Bounded by the plugin's fan-out budget (see [README](README.md#agents)).

**Output shape:** a unified report with an overall verdict, per-repo summaries and deltas, and cross-cutting pattern detection (the same issue recurring across repos).

### `cicd-expert` -- natural-language router

```
/cicd-expert "why is my deploy job hanging"
```

**What happens:** this is the fallback entry point. If your request clearly matches one workflow, the router dispatches straight to it; if it names 3+ repos or says "cross-project pipeline audit" or `--team`, it routes to `cross-project-pipeline-audit`; otherwise it asks a clarifying question before dispatching.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| The skill asks a clarifying question instead of running | Your request did not clearly match one of the eight named workflows | Answer the question, or invoke the specific skill directly (for example `/cicd-expert:review-pipeline` instead of the generic `/cicd-expert`) |
| A finding cites an action or platform feature you can't find in the docs | The agent invented CI-platform syntax instead of checking current documentation | Ask it to verify with the `context7` plugin (`resolve-library-id` + `query-docs`) before trusting the syntax; the agent should do this automatically when context7 is available |
| `cross-project-pipeline-audit` seems to do nothing, or produces no team-lead output | Dispatched via the plugin namespace, which silently strips the `Agent` tool the team lead needs to fan out | Dispatch `cicd-team-lead` as `general-purpose` with the agent file's body inlined as the prompt prefix (see the RUNTIME DISPATCH NOTE in `agents/cicd-team-lead.md`) |
| A self-hosted-runner finding calls a Namespace-style managed-ephemeral-runner service "self-hosted" | `references/self-hosted-runners.md` frames managed ephemeral runner services as a distinct vendor-class category, not full self-hosting | Re-run with `self-hosted-audit`, which applies the three-layer model explicitly; execution-only migration to a managed runner service is not full self-hosting |
| Emitted YAML fails to parse | Rare -- the agent's pre-return verification includes a syntax check, but manual edits after generation can break it | Run the CI platform's own linter (for example `actionlint` for GitHub Actions) before committing |
| The `context7` plugin is not installed | It is an optional dependency for live library/API documentation lookups | Install it from the Claude Code plugin marketplace, or proceed without it -- the agent still functions, with weaker guarantees against invented syntax |

## FAQ

**Does it modify my CI config directly?** Not without approval. Every workflow ends with "offer to apply" -- fixes, migrations, and new pipeline files are written only after you say yes.

**Which model runs this?** The `cicd-expert` and `cicd-team-lead` agents both run on the session model -- always the strongest available Claude Code model, whichever one that is. `cicd-team-lead` additionally dispatches conductor-selected executors (Sonnet 5 @ `xhigh` or Opus 4.8) for per-repo reconnaissance in team mode; judgment and severity verdicts are never delegated to them.

**Do I need the `context7` plugin?** No, but it is recommended. Without it, the agent relies on its training data for CI-platform syntax, which can be stale for newer features.

**Can it audit non-GitHub-Actions pipelines?** Yes -- GitLab CI/CD, Jenkins, Buildkite, and Tekton are covered by the same seven-lens framework and knowledge base; platform-specific syntax differences are documented in `references/platform-comparison.md`.
