# Multi-Agent Software Factory

Portable, harness-neutral instructions that turn **any** project into a coordinated multi-agent software factory.

Works with IDE agents (Cursor, Claude Code, Codex, Gemini CLI, and similar) and with self-hosted teammate / chat-bot apps that load `SKILL.md` recipes, project instruction files (`AGENTS.md` / `CLAUDE.md`), and can run tools in a workspace or sandbox.

Built on [mattpocock/skills](https://github.com/mattpocock/skills). This repo does not replace those skills; it installs them and adds an Orchestrator playbook with three entry points.

## Install (one line)

Paste this into your agent (with or without a project open):

```text
Follow the multi-agent software factory setup at https://raw.githubusercontent.com/nicnocquee/coding-agents/HEAD/SETUP.md and install it (into this project if one is open; otherwise as a harness preflight so the next project binds immediately).
```

- **Project open:** full install into that git tree (`docs/factory/`, router block, checks, smokes).
- **No project yet:** **harness preflight** — installs skills into the harness skill store, caches the playbook under the agent's factory home, and stops ready. When you later point at a repo, say: `Bind the multi-agent software factory to this project`.

For a local checkout instead of the raw URL, point at that clone's `SETUP.md` path.

## What you get

| Entry point | Say something like… | What happens |
| ----------- | ------------------- | ------------ |
| **Question** | "How does auth work?" | Read-only explorers; cited answer; no code changes |
| **Bug** | "Checkout fails when…" | Triage → trivial fix or hard diagnosis → review → PR |
| **Feature** | "Add saved searches" | Grill with you → spec/tickets → implementers → review → PR |

Human gates sit at alignment (grilling), ticket approval, hard-bug hypothesis ranking, and final PR review. Use the harness's native approval cards when available. Everything between those gates can run AFK when the harness supports subagents.

## Operating modes

The installer probes capabilities (subagents, parallelism, isolated workspaces) and picks one mode:

- **Full** — parallel subagents, isolated workspaces (git worktrees or sandbox checkouts), merger, parallel reviewers
- **Serial subagents** — same roles, one at a time
- **Single-agent sequential** — each role is a phase; cold review in a fresh session

Same flows and artifacts in every mode; only concurrency and isolation change. If subagents inherit parent chat, briefs force "ignore prior conversation."

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

After a **project** install, the target gains `docs/factory/`, a marker-delimited block in its instruction file, and Matt Pocock setup under `docs/agents/`. After **harness preflight**, skills and a cached playbook live on the agent; Bind copies them into the project when you open one.

## Upgrade / uninstall

- **Upgrade:** paste the same one-liner again. SETUP compares versions and refreshes only `docs/factory/` and the marked block.
- **Uninstall:** ask the agent to follow the Uninstall section of `SETUP.md`. Skills, glossary, and ADRs stay unless you ask to remove them.

## Design notes

- Harness-neutral: capability detection, not product-name adapters.
- Composes Matt's skills (`implement-spec`, `code-review`, `triage`, `diagnosing-bugs`, …) instead of reinventing them.
- Subagents are leaves; only the Orchestrator spawns. Prefer fresh context; degrade safely when the harness inherits chat. Communicate by context pointer.
