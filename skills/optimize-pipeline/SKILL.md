---
name: optimize-pipeline
description: |-
  This skill should be used when the user asks to "optimize my pipeline", "CI is slow", "reduce CI time", "pipeline takes too long", "speed up my builds", "CI cost too high", "reduce pipeline cost", "improve CI performance", "shorten feedback loop", "45 minute CI", or "queue time is killing us". Diagnoses whether the bottleneck is queue time, execution time, unnecessary serialization, or fleet capacity, then applies the optimization hierarchy (skip work > shorten critical path > cache smartly > fix fleet topology > tune toolchain). Produces before/after configuration diffs with expected impact.
argument-hint: '[optional: scope -- all / file path / specific pain point]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Optimize Pipeline

Dispatches the cicd-expert agent with an optimize-workflow briefing.

## Step 1: Gather current state

Before dispatching, collect:
1. Current wall-clock CI time (if user mentioned it)
2. Platform and runner type (hosted, self-hosted, ARC)
3. Repo topology (monorepo, polyrepo, size)
4. Read existing CI config files to identify the job graph

## Step 2: Dispatch cicd-expert

```
Agent({
  description: "Optimize CI/CD pipeline",
  subagent_type: "cicd-expert:cicd-expert",
  model: "fable",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: optimize

CURRENT STATE:
- Reported CI time: <if mentioned>
- Platform: <detected>
- Runner type: <detected>
- Repo topology: <detected>
- Working directory: <absolute path>
- CI config files: <list of files found>

DELIVERABLES:
1. Bottleneck diagnosis:
   - Separate queue time from execution time
   - Identify the critical path (longest chain of dependent jobs)
   - Identify unnecessary serialization (`needs:` edges that do not exchange artifacts)
   - Identify wasted work (jobs that run when they should be skipped)
   - Identify fleet capacity issues (many jobs waiting for few runners)

2. Optimization plan ordered by the hierarchy:
   a. Skip irrelevant work -- path filtering, affected-only execution, cancel superseded runs
   b. Shorten critical path -- remove unnecessary `needs:` edges, isolate merge gate
   c. Cache smarter -- dependency caches, Docker layer caches, artifact reuse (with hit rate estimate)
   d. Fix fleet topology -- move lightweight jobs off scarce runners, scale runners, warm pools
   e. Tune toolchain -- package manager, compiler cache, matrix size, Docker build strategy

3. For each optimization:
   - Expected time savings (estimated)
   - Confidence grade
   - Before/after YAML diff
   - Risk assessment (could this hide a real failure?)

4. Monorepo-specific (if applicable):
   - Graph-aware affected-only execution strategy
   - Stable gatekeeper job design
   - Remote caching viability
   - Merge queue integration

5. Cost analysis (if applicable):
   - Current compute cost drivers
   - Expected savings per optimization
   - Runner class efficiency (are expensive runners doing cheap work?)

6. Signal preservation check:
   - Does any optimization weaken merge safety?
   - Are we hiding failures behind retries or continue-on-error?

CONSTRAINTS:
- Apply the optimization hierarchy IN ORDER -- do not jump to toolchain tuning
- Measure before optimizing -- do not assume the bottleneck
- Every optimization must preserve signal quality -- no fake-green shortcuts
- Quantify expected impact where possible

Proceed with your standard workflow (reference files first -- especially 14, then analyze the pipeline DAG, then produce the plan).
```

## Step 3: Relay and apply

Present the bottleneck diagnosis + optimization plan ordered by impact. Offer to apply changes incrementally.

## Never do

- Skip directly to toolchain tuning without checking higher-leverage optimizations first
- Recommend `continue-on-error` as a "performance optimization"
- Claim savings without showing the reasoning
- Add emojis
