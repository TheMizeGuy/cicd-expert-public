---
name: migrate-pipeline
description: |-
  Migrate CI/CD pipelines between platforms or upgrade pipeline architecture. Supports Jenkins to GitHub Actions, Jenkins to GitLab CI, GitLab to GitHub Actions, CircleCI to GitHub Actions, Travis CI to GitHub Actions, Buildkite to GitHub Actions, and any other cross-platform migration. Also handles major version upgrades (reusable workflows adoption, composite actions refactoring, monorepo strategy changes). Triggers on "migrate from Jenkins to GitHub Actions", "move from GitLab CI to GitHub Actions", "convert my Jenkinsfile", "switch CI platforms", "migrate pipeline", "move from CircleCI", "upgrade my CI", "modernize my pipeline", "refactor pipeline architecture". Produces phased migration plan with feature parity mapping and rollback at each stage.
argument-hint: '[optional: source platform -> target platform, or migration scope]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Migrate Pipeline

Dispatches the cicd-expert agent with a migration-workflow briefing.

## Step 1: Identify migration scope

1. **Source platform**: Jenkins, GitLab CI, CircleCI, Travis CI, Buildkite, Tekton, or other
2. **Target platform**: GitHub Actions, GitLab CI, Jenkins, Buildkite, Tekton, or other
3. **Migration type**: full platform migration, partial (e.g., add GHA alongside Jenkins), or architecture upgrade within same platform
4. Read existing CI configuration files to catalog the source pipeline

If user provided source->target, proceed. If not, ask.

## Step 2: Dispatch cicd-expert

```
Agent({
  description: "Migrate CI/CD pipeline",
  subagent_type: "cicd-expert:cicd-expert",
  model: "opus",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: migration

SOURCE PLATFORM: <identified>
TARGET PLATFORM: <identified>
MIGRATION TYPE: <full / partial / architecture upgrade>
WORKING DIRECTORY: <absolute path>
SOURCE CONFIG FILES: <list of files found>

DELIVERABLES:
1. Source pipeline inventory:
   - All jobs/stages and their dependencies (DAG)
   - Triggers and events
   - Secrets and credentials used
   - Environments and deployment targets
   - Shared pipeline components (libraries, templates, plugins)
   - Runner/agent requirements (OS, capabilities, labels)
   - External integrations (notifications, artifact stores, registries)

2. Concept mapping table:
   | Source concept | Target equivalent | Notes |
   |---|---|---|
   | Jenkins stage | GitHub Actions job | ... |
   | Jenkins agent label | GitHub Actions runs-on | ... |
   | Jenkinsfile shared library | Reusable workflow / composite action | ... |
   | ... | ... | ... |

3. Feature parity assessment:
   | Feature | Source | Target | Gap / workaround |
   |---|---|---|---|
   For each feature: can the target platform do it natively, does it need a workaround, or is it unsupported?

4. Phased migration plan:
   Phase 1: <description> -- rollback: <how>
   Phase 2: <description> -- rollback: <how>
   Phase 3: <description> -- rollback: <how>
   ...
   Each phase:
   - What migrates
   - Parallel-run strategy (both old and new active?)
   - Verification criteria (how to confirm this phase is healthy)
   - Rollback procedure

5. Target configuration files:
   - Complete workflow/pipeline YAML for the target platform
   - With comments mapping back to source features
   - Following all security best practices (pinning, permissions, etc.)

6. Risk assessment:
   - Features that lose fidelity in migration
   - Secrets that need re-provisioning
   - Branch protection/rules that need reconfiguring
   - Team workflow changes required

CONSTRAINTS:
- Read platform-comparison.md for platform tradeoffs
- Map concepts accurately -- do not invent target-platform features
- Use context7 for target platform syntax verification
- Preserve all existing capabilities unless the user explicitly drops them
- Include rollback at EVERY phase -- no irreversible jumps
- If the source uses platform-specific features with no direct equivalent, flag as a gap with a workaround proposal

Proceed with your standard workflow (reference files first, then read all source CI config, then produce the migration plan).
```

## Step 3: Relay and apply

Present the concept mapping + phased plan. Offer to produce target configuration files for Phase 1.

## Never do

- Migrate without reading the full source pipeline first
- Produce target config that drops features silently
- Skip the rollback plan for any phase
- Add emojis
