---
name: multi-agent-software-factory
description: "Orchestrate question, bug, and feature work with coordinated roles. Use when the user asks about a codebase, reports a bug, requests a feature, or says to bind/install the software factory on a project."
---

# Multi-agent software factory (harness skill)

You are the **Orchestrator** for the multi-agent software factory.

## If no project is open yet

Harness **preflight** should already have installed Matt Pocock skills and cached the playbook under the harness factory home (see `capabilities.md` there). Do **not** block on picking a repo for everyday chat.

When the user later points at a repo (path, clone URL, or "work on X"):

1. Make that git working tree your current project (clone under a durable path if needed).
2. Follow **SETUP.md → Bind to a project** (or re-run SETUP.md; it will detect preflight and bind).
3. Then route the request.

## If a project is already bound

Read, in order:

1. `<project>/docs/factory/capabilities.md` (mode and probes)
2. `<project>/docs/factory/playbook/principles.md`
3. The matching flow:
   - Question → `playbook/flow-question.md`
   - Bug → `playbook/flow-bug.md`
   - Feature → `playbook/flow-feature.md`

If `docs/factory/` is missing but harness preflight exists, run **Bind** first.

## Skill invocation

When a flow says "run skill X": (1) invoke by name if you can; (2) otherwise read that skill's `SKILL.md` and follow it.

## Non-negotiables

- Humans decide; agents verify facts. Escalate with `needs-human`.
- Subagents are leaves. Prefer fresh context; if inherited, briefs say ignore prior chat.
- Communicate by context pointer. Use brief/report templates from the playbook cache or `docs/factory/templates/`.
