# GitHub Actions Gotchas

Non-obvious platform behaviors that surface as confusing CI failures.

## Merge Queue Gotchas

- **Dual `merge_group` + `push` events cause phantom concurrency-cancel failures.** A workflow
  triggered on both `merge_group` and `push` that shares one `concurrency:` group can have its
  merge-queue run canceled by an unrelated `push` event (or vice versa), producing a merge-queue
  failure that has nothing to do with the code under test. Scope the concurrency group to the event
  type (for example `group: ${{ github.workflow }}-${{ github.event_name }}-${{ github.ref }}`), or
  give `merge_group` runs their own group namespace.
- **Path filters are silently bypassed in merge queues.** `paths:` / `paths-ignore:` filters that work
  correctly on `pull_request` do not reliably skip a job the same way inside a `merge_group` run --
  the merge queue evaluates the merged commit, not the PR diff, so a required check can unexpectedly
  run (or fail to run) against a full-repo checkout. Do not assume a path-filtered job is a no-op in
  the queue; verify with an actual queued run before relying on it as a merge gate.

## Local Composite Action Gotchas

- **Order matters: reference a local composite action AFTER `actions/checkout`.** `GITHUB_WORKSPACE`
  starts empty; a composite action step that runs before checkout cannot find its own repo-relative
  path or any files it expects to read from the checked-out tree.
- **Composite actions cannot proxy actions with `pre:` / `post:` lifecycle hooks.** Wrapping an action
  that relies on a `post:` step (for example a runner-hardening action or a cache action) inside a
  local composite action drops the hook -- it either crashes with an unclear error or is silently
  skipped, so cache saves or cleanup steps never run. Call actions with lifecycle hooks directly from
  the workflow YAML, not through a composite action wrapper.
