# Skills manifest

Required skills from [mattpocock/skills](https://github.com/mattpocock/skills). Install **exactly one** route (plugin **or** skills.sh **or** raw copy). Installing plugin and skills.sh together duplicates every skill.

## Required skills

### Engineering

| Skill | Why the factory needs it |
| ----- | ------------------------ |
| `setup-matt-pocock-skills` | Per-repo tracker, labels, domain docs |
| `ask-matt` | Router map over the skills |
| `grill-with-docs` | Feature alignment + glossary/ADRs |
| `domain-modeling` | Glossary/ADR vocabulary |
| `codebase-design` | Deep-module / seam vocabulary |
| `triage` | Incoming bugs/requests → agent briefs |
| `diagnosing-bugs` | Hard-bug loop |
| `tdd` | Red-green implementation |
| `implement` | Single-ticket / small build |
| `implement-spec` | Task-graph frontier orchestration |
| `to-spec` | Publish a buildable spec |
| `to-tickets` | Tracer-bullet tickets + blocking edges |
| `code-review` | Standards + Spec review |
| `pr` | PR body shape |
| `research` | Background primary-source research |
| `prototype` | Throwaway runnable answers |
| `wayfinder` | Foggy / multi-session decision maps |
| `retro` | Environment improvements after a session |
| `improve-codebase-architecture` | Deepening survey when seams are missing |

### Productivity

| Skill | Why the factory needs it |
| ----- | ------------------------ |
| `grilling` | Interview primitive |
| `handoff` | Portable session handoff |
| `wait-what` | Re-pitch when a message does not land |
| `writing-for-agents` | Style guide for agent-facing docs |

### Explicitly excluded

- `skills/misc/` (frozen)
- `skills/in-progress/` (not shipped in the plugin). The `software-factory` skill there is vocabulary reference only for designers of factories; it is not required at runtime.

## Install routes (pick one)

Order of preference when the executing agent chooses a route. Use **exactly one** family; do not combine a managed plugin with skills.sh.

1. **Managed plugin** (if the harness supports it and updates itself):
   - Claude Code: `claude plugin install mattpocock-skills@claude-plugins-official`
   - Codex: `codex plugin marketplace add mattpocock/skills` then `codex plugin add mattpocock-skills@mattpocock`
   - GitHub Copilot: marketplace add + install per [mattpocock/skills README](https://github.com/mattpocock/skills)
2. **Gemini CLI** (copies; not auto-updating):
   - `gemini skills install https://github.com/mattpocock/skills.git --path skills/engineering`
   - `gemini skills install https://github.com/mattpocock/skills.git --path skills/productivity`
3. **skills.sh** (editable copy; manual updates) — preferred portable default for IDE CLIs:
   - `npx skills@latest add mattpocock/skills -a <agent>` (omit `-a` to choose; include `setup-matt-pocock-skills` when asked)
   - Update: `npx skills@latest update`, then re-run `add` for new skills
4. **Project `.agents/skills/`** (chat bots, teammate apps, and harnesses that load repo skills):
   - Clone or sparse-checkout `mattpocock/skills` into a temp dir, then copy each required skill folder into `<project>/.agents/skills/<name>/` (keep `SKILL.md` and any skill-local files).
   - Prefer this when `npx` / plugin CLIs are unavailable inside a sandbox or computer, but the agent can write the project tree.
5. **Harness skill store** (account- or space-level skills UI / import API):
   - Import each required skill's `SKILL.md` (and attachments if the product supports them) into the harness skill store so bots can invoke them by name.
   - Still keep a project copy under `.agents/skills/` when the repo is shared with IDE agents, so both surfaces see the same recipes.
6. **Raw fallback**: fetch each `SKILL.md` via GitHub raw/API into the skills folder the harness already reads.

Also install the factory harness skill from this repo's `factory/harness-skill/SKILL.md` as `multi-agent-software-factory` whenever a skill store (or global skills dir) is available. That skill tells the agent how to route work and how to **Bind** after a no-project preflight.

Done when `ask-matt` and `setup-matt-pocock-skills` are discoverable (skill store, plugin, or readable `SKILL.md` on disk) **and** every skill in the tables above is present. For harness preflight, also require `multi-agent-software-factory` to be discoverable.

## Skill invocation fallback (identical wording everywhere)

When a procedure says "run skill X":

1. If the harness can invoke a skill by name, do that.
2. Otherwise, locate and read that skill's `SKILL.md` (installed skills directory, or fetch from `https://github.com/mattpocock/skills` under `skills/engineering/<name>/SKILL.md` or `skills/productivity/<name>/SKILL.md`) and follow it.

Never invent a substitute procedure when the skill file is reachable.

## Raw URL pattern

```
https://raw.githubusercontent.com/mattpocock/skills/HEAD/skills/engineering/<skill>/SKILL.md
https://raw.githubusercontent.com/mattpocock/skills/HEAD/skills/productivity/<skill>/SKILL.md
```

Replace `HEAD` with a pinned commit SHA when you need a frozen install.
