# Principles

These rules bind every role and every operating mode. When a procedure conflicts with a principle, the principle wins.

## Humans decide, agents verify facts

- **Decisions** (product intent, scope, risk, merge) belong to the human.
- **Facts** (what the code does, whether a test fails, whether a claim reproduces) belong to the agent.
- Do not invent a decision when the human has not made one. Escalate with `needs-human` instead.

## Alignment before code

- Misalignment is the most common failure. Grill before you build.
- Update `GLOSSARY.md` and ADRs as decisions land so the next session does not relearn the jargon.
- Specs and tickets are behavioral contracts, not procedural scripts. Prefer interfaces and acceptance criteria over file paths and line numbers.

## Feedback loops before fixes

- For bugs: refuse to theorise until a **tight feedback loop** exists (one command that goes red on *this* bug).
- For features: agree the **test seams** before writing tests. Prefer existing seams.
- Automated checks (types, lint, tests) are a hard gate for every Implementer. Green is required before reporting done.

## Small deliberate steps

- Tracer-bullet vertical slices, not horizontal layer dumps.
- One ticket, one fresh context, one isolated workspace (when the mode allows).
- The rate of feedback is the speed limit.

## Context hygiene

- Every subagent should start with a **fresh context window**. Prefer harness options that do not fork the parent conversation into the child.
- If the harness **always inherits** parent chat into subagents, treat that as a degraded C1: still use subagents for parallelism, but put **only** the brief in the spawn message, tell the child to ignore prior chat, and prefer Serial when inherited noise causes wrong actions. Record `context: inherited` in `docs/factory/capabilities.md`.
- Communicate by **context pointer** (path, issue URL, commit SHA), not by pasting large copies of specs or diffs into prompts.
- Keep grilling → spec → tickets in **one unbroken context** when possible. Clear or compact only at a phase boundary.
- At a phase boundary, apply the decision tree in [coordination.md](coordination.md): continue, clear, handoff, subagent, or compact. Do not compact mid-phase.

## One orchestrator spawns everyone

- Only the **Orchestrator** talks to the human and spawns subagents.
- Subagents are **leaves** (depth 1). Nested spawning is the harness's job to forbid; this playbook states the intent once.
- Spec Writer and Ticket Planner are not subagents: they run in the Orchestrator's context because grilling is their primary source.

## Start slow

Accelerators without brakes make a slop cannon. Three brakes on every change flow:

1. **Automated checks**: deterministic gates every Implementer must pass.
2. **Automated review**: Standards + Spec axes, reported side by side, never merged or reranked.
3. **Human review**: a human reads the PR before merge. What it catches becomes new checks and sharper standards.

Open the throttle (parallel frontier, AFK implementers) only after the brakes earn trust.

## Skill invocation

When a procedure says "run skill X":

1. If the harness can invoke a skill by name, do that.
2. Otherwise, locate and read that skill's `SKILL.md` (installed skills directory, or the path recorded in `docs/factory/skills-manifest.md`) and follow it.

Never invent a substitute procedure when the skill file is reachable.
