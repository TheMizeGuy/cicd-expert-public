# Optimization Patterns

Cross-platform performance patterns for reducing queue time, compute waste, and feedback latency.

## The Optimization Hierarchy

Always apply in order. Do NOT jump to toolchain tuning first.

### 1. Skip irrelevant work

Path-based selection, affected-only execution, and duplicate-run cancellation usually save more than low-level tuning.

- GitHub: `paths` / `paths-ignore` filters, `concurrency.cancel-in-progress`
- GitLab: `rules: changes`, `interruptible: true`
- Monorepos: graph-aware affected-only execution (e.g., Turborepo, Nx, Bazel)

### 2. Shorten the critical path

Once required work is identified, remove unnecessary `needs` edges, isolate the merge gate, run independent jobs in parallel. DAG quality matters more than the number of jobs.

### 3. Keep hot state where it pays off

Use caches deliberately, not superstitiously:
- Language-native dependency caches (npm/pnpm/yarn, pip, Go modules)
- Docker layer caches (registry cache, `mode=max`, BuildKit mount caches)
- Artifact reuse (build once, verify many times)
- Cache keys should follow lockfiles and toolchain versions.
- Caches only help when keys are stable enough to hit, small enough to avoid thrash, and placed on data that is actually expensive to rebuild.

### 4. Address fleet topology

If jobs spend more time waiting than running, the bottleneck is fleet capacity:
- Scale runners or use warm pools
- Move lightweight orchestration jobs off scarce heavyweight machines
- Measure queue time separately from execution time

### 5. Optimize the toolchain

Only after the above:
- Package manager choice and settings (pnpm vs npm, frozen lockfile)
- Compiler caches (ccache, swc, esbuild)
- Matrix strategy sizing
- Docker build strategies (multi-stage, BuildKit)
- ARM64 runners (reported 7-8x speedup on generic builds)

## Monorepo Rules

| Rule | Practical implication |
|---|---|
| Prefer graph-aware affected execution | Only changed packages and their reverse dependencies should rebuild or retest |
| Keep a stable merge gate above selective execution | Required checks should still emit a durable status even when most leaf jobs are skipped |
| Separate dependency-manifest changes from source changes | Dependency scans and cache invalidation should key off lockfiles, not arbitrary source edits |
| Use remote caching only if hit rates stay high | Oversized or constantly invalidated caches increase cost without helping lead time |

## Cost Control Without Breaking Trust

| Practice | Effect |
|---|---|
| Cancel superseded runs | Stops paying for stale validation |
| Mark jobs interruptible where safe | Lets newer pipelines preempt obsolete work |
| Split blocking from advisory checks | Keeps merge-critical feedback fast |
| Keep heavyweight compute on the right runner class | Prevents cheap tasks from clogging expensive hosts |
| Avoid duplicate scans with different thresholds | Parse once, apply policy once |

## Self-Hosted Parallelism Warning

On a single scarce runner, splitting work into many jobs often creates fake parallelism: repeated checkout, repeated install, and more queueing. Collapsing lint/build/test into one heavy job often saves more time than cross-job parallelism when every job waits for the same runner.
