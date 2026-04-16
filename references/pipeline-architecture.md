# Pipeline Architecture and Quality Gates

Cross-platform design principles for CI/CD pipelines that stay fast, trustworthy, and reviewable.

## Core Design Rules

1. **Put repo-local validation first.** Formatting, linting, static checks, dependency resolution, unit tests, and a deterministic build. Cheapest checks, easiest to mandate.

2. **Fail early on merge-blocking defects.** If the workflow knows in minutes that code will not compile, do not spend thirty minutes building images or running browsers.

3. **Build once, verify many times.** A mature pipeline builds an artifact once per revision, then fans out verification around that artifact: smoke, integration, vulnerability scanning, provenance, deployment verification. Rebuilding in later environments turns the pipeline into a moving target.

4. **Keep trust boundaries visible.** Fork PR validation, deployment jobs, production credential access, and privileged self-hosted runners should not share the same assumptions. Pipeline design should make those boundaries obvious in triggers, permissions, and runner selection.

5. **Prefer composition over giant files.** Reusable workflows, components, templates, shared libraries, and task bundles are operationally safer than one mega-pipeline.

## Pipeline as a DAG

The common pattern across modern CI/CD systems is a versioned DAG of checks, artifact creation, promotions, and deployment controls. Model outcomes and trust boundaries, not just a `build -> test -> deploy` diagram.

- Use dependencies (`needs`, `stage`, task ordering) only where artifacts or trust boundaries actually require them.
- Critical-path time drops more from removing unnecessary serialization than from micro-optimizing individual steps.
- Separate fast validation gates from slower evidence-gathering.

## Recommended Baseline Gate Stack

| Layer | Typical contents | Default stance |
|---|---|---|
| Fast PR validation | format, lint, type-check, unit tests, dependency install sanity, small build | Blocking |
| Merge safety | required status check, branch protection/ruleset, merge queue compatibility, artifact presence | Blocking |
| Artifact integrity | package build, SBOM/provenance, signature where needed, publish to registry | Blocking for release paths |
| Broader confidence | integration tests, e2e tests, contract tests, environment smoke checks | Usually blocking on protected branches |
| Security and compliance | SAST, dependency review, secret scanning, IaC/container scanning | Risk-tier dependent |
| Production promotion | environment approvals, rollout analysis, deployment verification | Blocking |
| Slow evidence | long soak tests, fuzzing, exploratory scans, synthetic canaries | Advisory or scheduled |

The best default is not "make every job blocking." It is "make the smallest trustworthy set of checks blocking, and make the rest explicit as advisory, scheduled, or release-only."

## Merge Gate Design

- Make the merge gate explicit and stable.
- Branch protection/rulesets should depend on a small set of durable required checks, not a shifting list of ad hoc jobs.
- Put human approvals at promotion boundaries (environment risk, separation of duties), not as generic friction after every build.

## Artifact Promotion

- Treat artifact creation and artifact promotion as separate concerns.
- The build creates an immutable artifact once; later stages promote and verify that artifact instead of rebuilding.
- Measure pipeline quality with outcome metrics (DORA), not just job duration.
