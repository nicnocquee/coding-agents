# Roles

Each role has one purpose, one (or two) backing skills, a defined input brief, an output contract, and a permission set. Permissions are hard: a role that is read-only must not write product code.

Templates: [../templates/brief.md](../templates/brief.md), [../templates/report.md](../templates/report.md).

## Permission legend

| Code | Meaning |
| ---- | ------- |
| `read` | Read repo, docs, tracker |
| `write-notes` | Write exploration or research notes (outside the repo unless the project says otherwise) |
| `write-docs` | Write `GLOSSARY.md`, ADRs, agent briefs, specs, tickets |
| `write-code` | Edit product source and tests |
| `git` | Commit, branch, worktree or sandbox checkout, merge (within its brief) |
| `spawn` | Start subagents |
| `talk-human` | Ask the human questions |

## Orchestrator

- **Purpose:** Classify the request (question, bug, feature), hold alignment context, spawn and aggregate subagents, enforce gates and caps.
- **Backing skills:** routing from `ask-matt`; orchestration pattern from `implement-spec`; phase boundaries from `ask-matt`'s phase-boundary rules.
- **Inputs:** the human's request; `docs/factory/capabilities.md`; playbook flows.
- **Output:** a completed flow report to the human (mode used, artifacts, next step).
- **Permissions:** `read`, `write-docs`, `spawn`, `talk-human`. Never writes product code.
- **May spawn:** yes (only role that may).

## Explorer

- **Purpose:** Map a scoped area of the codebase for another role.
- **Backing skills:** none required; follow read-only exploration discipline.
- **Inputs:** brief with scope pointers (glossary, ADRs, directories or concepts).
- **Output:** markdown notes outside the repo (path recorded in the report), under 400 words of summary plus pointers.
- **Permissions:** `read`, `write-notes`.
- **May spawn:** no (leaf).

## Researcher

- **Purpose:** Investigate a question against primary sources and capture findings.
- **Backing skill:** `research`.
- **Inputs:** the question; where to save the findings file.
- **Output:** one Markdown findings file with citations; report pointer.
- **Permissions:** `read`, `write-notes` (and write the findings file where the project stores research).
- **May spawn:** no (leaf). Prefer background when the mode allows.

## Triager

- **Purpose:** Move an incoming bug or request through triage roles; verify claims; emit an agent brief when ready.
- **Backing skill:** `triage` (and its agent-brief format).
- **Inputs:** issue or natural-language report; tracker config; `.out-of-scope/`.
- **Output:** category + state role, optional agent brief comment, triage notes.
- **Permissions:** `read`, `write-docs` (tracker comments and labels only during triage). No product code.
- **May spawn:** no (leaf).

## Diagnoser

- **Purpose:** Hard-bug diagnosis: build a feedback loop, rank hypotheses, instrument, fix with a regression test at a correct seam.
- **Backing skill:** `diagnosing-bugs`.
- **Inputs:** confirmed repro (or report that resists first glance); glossary and ADRs in the area.
- **Output:** root cause, ranked hypotheses (for the human checkpoint), fix + regression test (or "no correct seam" finding), cleanup confirmation.
- **Permissions:** `read`, `write-code`, `git` (on a dedicated branch). Tagged debug logs only; remove before done.
- **May spawn:** no (leaf).

## Spec Writer and Ticket Planner

- **Purpose:** Turn an aligned conversation into a published spec and a task graph of tracer-bullet tickets.
- **Backing skills:** `to-spec`, `to-tickets`.
- **Inputs:** grilling outcome (primary source in the Orchestrator's context); codebase exploration.
- **Output:** published spec (`ready-for-agent`); tickets with blocking edges and acceptance criteria.
- **Permissions:** `read`, `write-docs`.
- **May spawn:** no. **Not a subagent:** runs in the Orchestrator's context so grilling stays a primary source.

## Implementer

- **Purpose:** Build one ticket (or one small change) test-first.
- **Backing skills:** `implement`, `tdd`.
- **Inputs:** brief pointing at the ticket (and parent spec); notes path if Explorer ran; glossary and ADRs.
- **Output:** commits on its branch; report with evidence (checks run); branch tip includes a merge of the current integration branch before "done" when using isolated workspaces.
- **Permissions:** `read`, `write-code`, `git`.
- **May spawn:** no (leaf). Fresh context when the harness allows; one ticket per agent.

## Merger

- **Purpose:** Land an Implementer's branch onto the integration branch; resolve conflicts or report `needs-human`.
- **Backing skills:** none; git operations only.
- **Inputs:** implementer branch, integration branch, ticket pointer.
- **Output:** merge commit (or conflict report); updated frontier signal for the Orchestrator.
- **Permissions:** `read`, `git`. No feature work.
- **May spawn:** no (leaf).

## Reviewer pair (Standards + Spec)

- **Purpose:** Two-axis review of the diff since a fixed point.
- **Backing skill:** `code-review`.
- **Inputs:** fixed point (commit/branch); standards sources + smell baseline; originating issue/spec.
- **Output:** two separate reports under `## Standards` and `## Spec`, never merged or reranked; one-line counts per axis.
- **Permissions:** `read` only. Leaves.
- **May spawn:** the Orchestrator (or the `code-review` skill runner) starts both leaf reviewers in parallel when the mode allows. The reviewers themselves do not spawn.

## PR Author

- **Purpose:** Shape the pull request body.
- **Backing skill:** `pr`.
- **Inputs:** branch diff; evidence of before/after; door call (one-way or two-way).
- **Output:** PR body (summary visual, evidence, merge danger).
- **Permissions:** `read`, `git` (open/update PR metadata only).
- **May spawn:** no (leaf).

## Retro agent

- **Purpose:** After a session (especially a bug or a sideways build), propose improvements to the **agent environment**, not product code.
- **Backing skill:** `retro` (writing style via `writing-for-agents`).
- **Inputs:** session log or current session; existing checks and standards.
- **Output:** ranked candidates (navigation pointers, automated checks, coding standards, tooling).
- **Permissions:** `read`, `write-docs` (proposals only unless the human asks to apply them).
- **May spawn:** no (leaf).

## Architect (on demand)

- **Purpose:** Survey deepening opportunities when the codebase blocks agents (for example: no correct test seam for a bug).
- **Backing skill:** `improve-codebase-architecture` (vocabulary from `codebase-design`).
- **Inputs:** area of concern; glossary and ADRs.
- **Output:** candidate list; picking one generates an idea that re-enters the feature flow at grilling.
- **Permissions:** `read`, `write-docs` (report only unless the human starts a feature flow).
- **May spawn:** no (leaf) unless the skill itself fans out explorations under Orchestrator control.

## Role → skill quick map

| Role | Skill(s) |
| ---- | -------- |
| Orchestrator | `ask-matt`, `implement-spec` (pattern) |
| Explorer | (none) |
| Researcher | `research` |
| Triager | `triage` |
| Diagnoser | `diagnosing-bugs` |
| Spec Writer / Ticket Planner | `to-spec`, `to-tickets` |
| Implementer | `implement`, `tdd` |
| Merger | (none) |
| Reviewer pair | `code-review` |
| PR Author | `pr` |
| Retro | `retro`, `writing-for-agents` |
| Architect | `improve-codebase-architecture`, `codebase-design` |

Alignment helpers used by the Orchestrator (not separate spawnable roles): `grill-with-docs`, `grilling`, `domain-modeling`, `prototype`, `wayfinder`, `handoff`, `wait-what`.
