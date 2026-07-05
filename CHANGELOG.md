# Changelog

All notable changes to `cicd-expert` are documented here.

## [0.2.1] - 2026-07-05

### Changed

- **Version realignment**: reset this public mirror's version from 0.2.2 to 0.2.1 so it tracks the private source of truth (`TheMizeGuy/cicd-expert`). Public mirror versions now move in lockstep with the private plugin; the earlier independent public-milestone numbering (0.2.2) is retired. No functional or content change — this bump only aligns the version number.

## 0.2.2 -- 2026-07-05

- Un-pinned both agents' `model: fable` frontmatter; `cicd-expert` and `cicd-team-lead` now inherit the session model (always the strongest available Claude) instead of a named model, so the plugin keeps working across model transitions.
- Rewrote every "Fable 5" reference in agent descriptions, skill bodies, and README to session-model language; dropped the `fable` keyword from `plugin.json`.
- Added a shared `references/dispatch-contract.md` (Agent call template, briefing conventions, execution mode, acceptance criteria) and pointed all seven single-workflow skills at it instead of each repeating the same dispatch boilerplate.
- Externalized the long briefing bodies of `audit-pipeline-security` and `self-hosted-audit` into skill-local `references/security-checklist.md` and `references/audit-briefing.md`, shrinking both `SKILL.md` files while keeping the full briefing content available.
- `cicd-expert` agent: added a worked finding-format example (severity + location + lens + graded evidence + runnable fix) and a 5-point pre-return verification checklist.
- Added an "Execution mode" note (via the dispatch contract) clarifying that dispatched agents may run inline in the main context instead of being dispatched, when the session model is already the strongest tier.
- Docs refresh: added this changelog and `USAGE.md` (quickstart, one worked walkthrough per skill, troubleshooting table, FAQ).

## 0.2.1 -- 2026-07-04

- Set the plugin author to TheMizeGuy with the `ben@meipath.com` contact.
- Brought the public mirror to parity with a private-side audit wave; scrubbed residual vault-note references from skill briefing text.

## 0.2.0 -- 2026-07-03

- Fixed the `cicd-expert` router skill's dispatch path.
- Added the two-tier team mode (`cicd-team-lead` agent, `cross-project-pipeline-audit` skill).
- Rewrote agent/skill descriptions in third person.
- Filled gaps in the `references/` knowledge base.

## 0.1.0 -- 2026-04-16

- Initial public release: 2 agents (`cicd-expert`, `cicd-team-lead`), 9 skills, and the embedded `references/` knowledge base (pipeline architecture, security hardening, optimization patterns, self-hosted runners, deployment/delivery, metrics/governance, platform comparison, review checklist, GitHub Actions gotchas).
