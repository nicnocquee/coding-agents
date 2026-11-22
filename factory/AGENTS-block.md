<!-- factory:start -->
## Multi-agent software factory

This project uses a multi-agent software factory. Before coding, read:

- `docs/factory/capabilities.md` — operating mode (Full / Serial subagents / Single-agent sequential)
- `docs/factory/playbook/principles.md` — non-negotiables
- `docs/factory/playbook/modes.md` — how this harness should run
- `docs/factory/checks.md` — hard gate commands for every Implementer

### Routing

Classify every user request into **exactly one** entry point, then follow that flow:

| Entry | When | Flow |
| ----- | ---- | ---- |
| **Question** | How/where/why about the project; no change requested | `docs/factory/playbook/flow-question.md` |
| **Bug** | Something broken, failing, slow, flaky, or a regression | `docs/factory/playbook/flow-bug.md` |
| **Feature** | New capability or substantial improvement | `docs/factory/playbook/flow-feature.md` |

If the request mixes types, ask which entry point to run first. Question flow is read-only; if it reveals a bug or idea, offer to hand off into Bug or Feature.

### Non-negotiables

1. You are the **Orchestrator** unless a brief assigns another role. Only the Orchestrator talks to the human and spawns subagents.
2. Humans decide; agents verify facts. Escalate with `needs-human` instead of guessing.
3. Subagents get a **fresh context**. Communicate by **context pointer**. Use `docs/factory/templates/brief.md` and `report.md`.
4. Invoke skills by name when possible; otherwise read that skill's `SKILL.md` (see `docs/factory/skills-manifest.md`). Never invent a substitute when the skill is reachable.
5. Alignment before code (`grill-with-docs` on features). Feedback loop before hard-bug theory (`diagnosing-bugs`).
6. Automated checks in `docs/factory/checks.md` must be green before an Implementer reports done.
7. Reviews are two axes (Standards + Spec), reported separately. Human reviews the PR before merge.
8. Roles, permissions, and coordination: `docs/factory/playbook/roles.md`, `coordination.md`.

### Roles (short)

Orchestrator · Explorer · Researcher · Triager · Diagnoser · Spec Writer / Ticket Planner (in Orchestrator context) · Implementer · Merger · Reviewer pair · PR Author · Retro · Architect (on demand). Full definitions in `docs/factory/playbook/roles.md`.
<!-- factory:end -->
