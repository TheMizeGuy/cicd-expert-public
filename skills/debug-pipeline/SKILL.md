---
name: debug-pipeline
description: |-
  This skill should be used when the user asks to "debug my pipeline", "fix my workflow", or reports "CI is broken", "pipeline failing", "workflow not running", "build failed", "deploy failed", "runner not picking up jobs", "workflow trigger not working", "secrets not available", "CI flaky", "pipeline keeps failing", or "why did CI fail". Systematically debugs CI/CD pipeline failures -- build errors, test failures, deployment issues, runner problems, trigger mismatches, permission errors, secret availability, flaky tests, and silent failures -- using systematic debugging (reproduce, isolate, hypothesize, test, confirm, fix, verify), never guessing. Produces root cause analysis with a concrete fix.
argument-hint: '[optional: error message, failing job name, or workflow file]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Debug Pipeline

Dispatches the cicd-expert agent with a debug-workflow briefing.

## Step 1: Gather failure context

Before dispatching, collect:
1. Error message or symptom (from user or from reading logs)
2. Which workflow file and job
3. When it started failing (after which change?)
4. Whether it is intermittent (flaky) or consistent
5. Platform and runner type

Read the failing workflow file and any recent git changes to CI config.

## Step 2: Dispatch cicd-expert

Dispatch per the shared contract `../../references/dispatch-contract.md`: `subagent_type:
"cicd-expert:cicd-expert"`, model omitted (inherits the session model; inline execution allowed
per the contract's Execution mode when the session model is already the strongest tier),
description "Debug CI/CD failure", prompt = the briefing below.

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: debug

FAILURE CONTEXT:
- Error message/symptom: <from user or logs>
- Workflow file: <path>
- Failing job: <name if known>
- Intermittent or consistent: <if known>
- Recent changes: <git log of CI-relevant changes>
- Platform: <detected>
- Runner type: <detected>
- Working directory: <absolute path>

DELIVERABLES:
1. Systematic diagnosis:
   a. Reproduce -- confirm the failure exists and identify exact conditions
   b. Isolate -- which job, which step, which command fails?
   c. Check common failure categories:
      - Trigger mismatch (event type, branch filter, path filter; `schedule` triggers auto-disable after 60 days of repo inactivity on public repos)
      - Permission error (GITHUB_TOKEN scope, environment protection)
      - Secret unavailability (fork PR, missing environment, wrong scope level; fork PRs get no secrets and a read-only GITHUB_TOKEN by default)
      - Runner issue (label mismatch, capacity, ARC scaling, ephemeral cleanup)
      - Dependency issue (cache miss, lockfile drift, registry outage)
      - Configuration syntax (YAML indentation, expression syntax, matrix)
      - Concurrency conflict (concurrent runs, resource contention; concurrency group names are case-insensitive, and each group allows at most one running + one pending run)
      - Network issue (egress, DNS, private network)
   d. Hypothesize -- form 1-3 ranked hypotheses based on evidence
   e. Test -- check each hypothesis against the config and logs
   f. Confirm -- identify root cause with evidence

2. Root cause analysis:
   - What failed
   - Why it failed
   - When it started failing (if determinable)
   - What change caused it (if determinable)

3. Fix:
   - Exact configuration change (YAML diff)
   - Why this fixes it (with confidence grade -- `[P]`/`[S]`/`[PxN]`/`[V]`/`[recall]`)
   - How to verify the fix

4. Prevention:
   - What would have caught this earlier?
   - Any workflow scanning or validation to add?

CONSTRAINTS:
- NEVER guess the root cause -- follow the systematic path
- Read the actual workflow file and any referenced reusable workflows -- env vars and secrets are NOT auto-propagated from caller to callee; each nesting level must re-pass secrets explicitly
- Check for common gotchas: YAML quoting, expression syntax, action version mismatches
- If the error involves a third-party action, check its documentation via context7 or WebFetch
- Document the root cause and fix clearly so the user can prevent recurrence

Proceed with your standard workflow (reference files first for prior similar failures, then read the pipeline config, then diagnose systematically).
```

## Step 3: Relay findings

Check the contract's acceptance criteria first. Present root cause + fix. Offer to apply the fix directly.

## Never do

- Guess the root cause without evidence
- Suggest "try this and see" without a hypothesis
- Skip reading the actual workflow file
- Add emojis
