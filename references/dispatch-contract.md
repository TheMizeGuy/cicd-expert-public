# Dispatch contract -- shared by all cicd-expert skills

Canonical dispatch mechanics for every skill in this plugin that dispatches the `cicd-expert`
agent (`design-pipeline`, `review-pipeline`, `optimize-pipeline`, `debug-pipeline`,
`audit-pipeline-security`, `migrate-pipeline`, `self-hosted-audit`, and the router's ambiguous
fallback). `cross-project-pipeline-audit` has its own dispatch (team-lead body inlined into a
`general-purpose` agent) and does NOT use this template.

## Agent call template

```
Agent({
  description: "<workflow>: <scope>",
  subagent_type: "cicd-expert:cicd-expert",
  // omit model -- inherits the session model (always the strongest available Claude)
  prompt: "<the invoking skill's briefing>"
})
```

Never dispatch `cicd-team-lead` directly by `subagent_type` -- plugin-namespaced dispatch strips
the `Agent` tool it needs for sub-dispatch. Route cross-project scope through the
`cross-project-pipeline-audit` skill instead.

## Briefing conventions (every briefing observes these)

- Open with `ORIGINAL USER REQUEST: <verbatim>` -- never paraphrase the user.
- State `WORKFLOW: <design | review | optimize | debug | security | migration | self-hosted>`.
- State `WORKING DIRECTORY: <absolute path>` -- never relative; list resolved config files.
- Carry the invoking skill's DELIVERABLES and CONSTRAINTS blocks verbatim.
- Close with: `Proceed with your standard workflow (reference files first -- especially <the
  invoking skill's most relevant reference file(s)> -- then read the CI config, then produce the
  output).`

## Execution mode

Dispatched agents inherit the session model -- always the strongest available Claude. If the
session model is already the strongest tier and the task is important or complicated, run the
skill's workflow inline in the main context (foreground) instead of dispatching: perform the
agent's mandatory grounding (reference files, project CI config) and the briefing's deliverables
directly. Never block on, or call out to, a specific unavailable model.

## Acceptance criteria (check every one before relaying a result)

1. Output follows the agent's report-format contract (Summary / Context / Findings / ... /
   Verification steps / References).
2. Non-obvious claims carry confidence grades (`[P]`/`[S]`/`[PxN]`/`[V]`/`[recall]`).
3. Every CRITICAL/HIGH finding includes a runnable YAML/config fix, not prose advice.
4. Verification steps the user can run are present.
5. Self-hosted or hybrid claims distinguish the three layers (execution/control/deployment)
   explicitly rather than collapsing them.

Any criterion failing: one re-dispatch naming the concrete gap. On a second failure, take the
work over inline rather than dispatching a third time.
