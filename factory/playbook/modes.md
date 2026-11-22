# Operating modes

Modes are selected from **capabilities**, not product names. SETUP records the result in `docs/factory/capabilities.md`. The Orchestrator reads that file at the start of every flow.

## Capabilities (reference)

| Id | Meaning |
| -- | ------- |
| C1 | Spawn a subagent with a fresh context and get a report back |
| C2 | Run subagents in the background or in parallel |
| C3 | Shell, git, and git worktrees |
| C4 | Invoke a skill by name (else read `SKILL.md`) |
| C5 | Ask the human a structured question (else plain chat) |
| C6 | Which instruction file the harness reads |

## Mode selection

| Mode | Required | Behavior in one line |
| ---- | -------- | -------------------- |
| **Full** | C1 + C2 + C3 | Parallel subagents; Implementers in worktrees; Merger; parallel Reviewers |
| **Serial subagents** | C1, missing C2 and/or C3 | Same roles, one at a time; single branch or sequential worktrees |
| **Single-agent sequential** | no C1 | Each role is a phase in one agent; cold Reviewer pass in a fresh session |

If C1 and C3 hold but C2 does not → **Serial**. If C1 and C2 hold but C3 does not → **Serial** (no worktrees; still one branch). Only Full when all three are true.

C4 and C5 affect *how* skills and questions run, not which mode is chosen. C6 selects which file receives the factory router block.

## Full

- Orchestrator fans out Explorers, Implementers, Merger, Reviewer pair.
- Implementers: one worktree + branch per ticket; merge integration tip before reporting done.
- Merger lands branches onto the integration branch.
- Reviewer: Standards and Spec in parallel leaves.
- Concurrency cap 3–4 on the Implementer frontier unless the human raises it.

## Serial subagents

- Same role contracts as Full.
- Spawn at most one working subagent at a time (plus optional background Researcher if C2 is partially available; if unsure, serialise everything).
- Prefer one integration branch; optional sequential worktrees if C3 exists without C2.
- Reviewer pair still runs as two cold passes when possible (even if sequential).

## Single-agent sequential

- No subagent spawn. The same agent plays each role as a **phase**.
- At each phase boundary, apply the tree in [coordination.md](coordination.md).
- Between tickets: **clear** or **handoff** so the next ticket starts clean; do not carry implementation noise into the next ticket.
- **Cold Reviewer pass (required default):** start a fresh session (or clear) with *only* context pointers (diff command, fixed point, spec path, standards paths). Run Standards then Spec. Do not review in the same hot context that wrote the code.
- **When the cold Reviewer pass may be skipped:** only for throwaway probes the human marked disposable (for example a `prototype` detour). **Never skip** for:
  - bug fixes that touch shared modules
  - any change destined for a PR into the default branch
  - any ticket on an integration branch

## Skill invocation in every mode

1. If C4: invoke the skill by name.
2. Else: open the skill's `SKILL.md` from the installed skills location (or the raw URL listed in `docs/factory/skills-manifest.md`) and follow it.

## Flow degradation summary

| Flow | Full | Serial | Single-agent |
| ---- | ---- | ------ | ------------ |
| Question | Parallel Explorers | Sequential Explorers | Explorer phases |
| Bug | Diagnoser/Implementer/Reviewers as subagents | Same, one at a time | Phases + cold review (never skip on shared-code fixes) |
| Feature | Parallel frontier + Merger | Sequential Implementers | One ticket per phase; clear between; cold review |

## Changing mode

Re-run SETUP.md (or re-run only capability detection) when the harness changes. Do not silently upgrade to Full without re-verifying C1–C3 with a smoke probe.
