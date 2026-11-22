# Multi-Agent Software Factory

Portable, harness-neutral instructions that turn **any** project into a coordinated multi-agent software factory. Works with Cursor, Claude Code, Codex, Gemini CLI, Grok-based agents, or anything else that can read a markdown file and follow it.

Built on [mattpocock/skills](https://github.com/mattpocock/skills). This repo does not replace those skills; it installs them and adds an Orchestrator playbook with three entry points.

## Install into a project (one line)

Open the project you want to instrument. Paste this into your coding agent:

```text
Follow the multi-agent software factory setup at https://raw.githubusercontent.com/nicnocquee/coding-agents/HEAD/SETUP.md and install it into this project.
```

The agent will detect its own capabilities, ask for approval, install Matt Pocock's skills, configure the issue tracker, copy the factory playbook into `docs/factory/`, and run smoke tests.

For a local checkout instead of the raw URL, point at that clone's `SETUP.md` path.

## What you get

| Entry point | Say something like… | What happens |
| ----------- | ------------------- | ------------ |
| **Question** | "How does auth work?" | Read-only explorers; cited answer; no code changes |
| **Bug** | "Checkout fails when…" | Triage → trivial fix or hard diagnosis → review → PR |
| **Feature** | "Add saved searches" | Grill with you → spec/tickets → implementers → review → PR |

Human gates sit at alignment (grilling), ticket approval, hard-bug hypothesis ranking, and final PR review. Everything between those gates can run AFK when the harness supports subagents.

## Operating modes

The installer probes capabilities (subagents, parallelism, git worktrees) and picks one mode:

- **Full** — parallel subagents, worktrees, merger, parallel reviewers
- **Serial subagents** — same roles, one at a time
- **Single-agent sequential** — each role is a phase; cold review in a fresh session

Same flows and artifacts in every mode; only concurrency changes.

## Repo layout

```text
SETUP.md                 # Agent-executable installer (the instruction)
README.md                # This file
factory/
  VERSION                # Payload version
  AGENTS-block.md        # Router block inserted into AGENTS.md / CLAUDE.md
  skills-manifest.md     # Required skills + install routes
  playbook/              # Principles, roles, flows, coordination, modes
  templates/             # Brief, report, handoff
```

After install, the **target project** gains `docs/factory/` (copy of the playbook), a marker-delimited block in its instruction file, and the usual Matt Pocock setup under `docs/agents/`.

## Upgrade / uninstall

- **Upgrade:** paste the same one-liner again. SETUP compares versions and refreshes only `docs/factory/` and the marked block.
- **Uninstall:** ask the agent to follow the Uninstall section of `SETUP.md`. Skills, glossary, and ADRs stay unless you ask to remove them.

## Design notes

- Harness-neutral: capability detection, not product-name adapters.
- Composes Matt's skills (`implement-spec`, `code-review`, `triage`, `diagnosing-bugs`, …) instead of reinventing them.
- Subagents are leaves; only the Orchestrator spawns. Fresh context every time. Communicate by context pointer.
