# Coordination

How agents talk, when to split context, how to fail, and how to avoid double work.

## Subagent rules

1. **Fresh context every time.** Do not inherit the parent's conversation.
2. **Orchestrator is the only spawner.** Subagents are leaves (depth 1). Nested spawning is out of scope for this playbook; the harness should enforce depth.
3. **Background when independent; foreground when the next step needs the result now.**
4. **Sparse prompts.** Prefer context pointers over pasted blobs.
5. **Concurrency cap** on the Implementer frontier: start at **3–4**. Lower when merge conflicts or cost dominate.

## Brief (in) and Report (out)

Use the templates in `docs/factory/templates/` (installed from this payload).

### Brief (Orchestrator → subagent)

Must include:

- **Goal** (one sentence)
- **Pointers**: issue/spec/ticket URLs or paths, notes directory, `GLOSSARY.md`, ADRs, `docs/factory/checks.md`
- **Acceptance criteria** (testable)
- **Out of scope**
- **Role permissions** (copy from [roles.md](roles.md))
- **Mode** (Full / Serial / Single-agent) so the subagent knows whether worktrees exist

Do **not** include stale file paths or line numbers as requirements. Prefer interfaces and behaviors (see triage agent-brief principles).

### Report (subagent → Orchestrator)

Under **400 words**:

- **Status:** `done` | `blocked` | `needs-human`
- **What changed** (commits, ticket ids)
- **Evidence** (commands run + outcomes)
- **Open questions**
- **Artifact pointers**

If `needs-human`, state the exact decision required. Do not guess.

## Phase-boundary tree

A **phase** ends when the work chunk is done (grilling, implementation, QA). Decide **only at the boundary**, top to bottom; the first yes wins:

1. **Continue?** Next phase needs this as a primary source, and enough smart-zone (~150k tokens) remains → stay.
2. **Clear?** Everything here is disposable for what comes next → empty the window.
3. **Handoff?** Need a portable file for a new harness, new directory, colleague, or mid-phase side task → write handoff markdown (OS temp dir unless the project defines otherwise).
4. **Subagent?** Scoped tightly enough to run AFK → spawn and keep this session untouched.
5. **Otherwise, compact** with an instruction so the summary keeps what the next phase needs.

Do **not** compact mid-phase. Mid-phase: continue, or split remaining work into subagents.

Primary source (full session) beats secondary source (summary). Pay the lossiness only when staying costs more than it saves.

## Escalation

- Subagents report `needs-human` instead of inventing product decisions.
- Orchestrator **batches** questions into one round to the human.
- A run never waits silently: it **ends**. A later human action (reply, label change, approval) starts a new run.
- Use `wait-what` when a message did not land; re-pitch with glossary vocabulary.

## Failure handling

| Failure | Action |
| ------- | ------ |
| Checks red after Implementer | One fresh Implementer on the same brief |
| Same ticket fails twice | Block ticket; treat brief as defective; escalate |
| Merge conflict | Merger attempts once; then `needs-human` |
| Cannot reproduce bug | `needs-info`; stop |
| Cap hit (bug: 3 attempts) | Handoff to human; stop looping |
| Review findings | One Implementer fixes all in one pass |

## Idempotency and ownership

- One **owner** per ticket. Track with `in-progress` (or tracker equivalent).
- Never start a ticket already `in-progress` under another owner.
- Re-running a flow must be safe: read tracker + git state first; skip work already done.
- Durable shared memory is only: issue tracker, git, `GLOSSARY.md`, `docs/adr/`, `.out-of-scope/`, and the notes directory. Agents have no other memory between runs.

## State locations

| Artifact | Where |
| -------- | ----- |
| Specs, tickets, briefs, labels | Issue tracker (or `.scratch/` if local) |
| Branches, PRs, commits | Git host |
| Domain language | `GLOSSARY.md` |
| Hard decisions | `docs/adr/` |
| Rejected enhancements | `.out-of-scope/` |
| Exploration notes | Outside the repo (path in `docs/factory/capabilities.md` or the brief) |
| Factory config | `docs/factory/` |

## Triage vocabulary

Canonical state roles: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. Category roles: `bug`, `enhancement`. Actual label strings live in `docs/agents/triage-labels.md` after project setup.
