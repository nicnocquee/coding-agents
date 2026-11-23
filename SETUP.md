# SETUP.md — Multi-Agent Software Factory

**You are an agent.** Execute this file to install the factory. Do not invent steps. Do not skip done-conditions.

## Contract

1. **Read this entire file first**, then explore read-only, then present a plan, then wait for approval before writing.
2. **Ask before writing.** Step 0 ends at a human gate. No installs or file edits before approval (except read-only probes).
3. **Never overwrite user content** outside factory markers, `docs/factory/`, the harness factory home, and skill-store entries you created for this factory.
4. **Every write is idempotent.** A second run at the same `VERSION` must produce no meaningful diff.
5. **Stop and report** on any failed done-condition after at most one fix attempt for that step.
6. **Payload location.** The installable payload lives next to this file under `factory/`. Resolve it as:
   - If this `SETUP.md` was opened by path → use `<directory-containing-SETUP.md>/factory/`.
   - If this `SETUP.md` was fetched by URL → fetch sibling paths under the same base URL (`factory/VERSION`, `factory/playbook/*`, `factory/harness-skill/*`, etc.).
7. **Skill invocation fallback** (use this wording everywhere): when a procedure says "run skill X": (1) if the harness can invoke a skill by name, do that; (2) otherwise locate and read that skill's `SKILL.md` and follow it. Never invent a substitute when the skill file is reachable.

---

## Step 0 — Detect (read-only)

### 0.0 Choose install scope

Inspect the current working directory and any path the human already named.

| Scope | When | What this run does |
| ----- | ---- | ------------------ |
| **Project** | cwd (or a named path) is a git working tree the agent can write | Full install into that project (Steps 1–6) |
| **Harness preflight** | no writable git project is open or named | Install skills + cache playbook at the **harness** so the next pointed-at project binds immediately (Steps H1–H5). Do **not** stall waiting for a repo pick unless the human insists. |
| **Bind** | harness preflight already exists **and** the human now points at a project (or cwd became a git tree) | Skip skill re-install when skills are present; run Steps 2–6 into that project |

**Default when ambiguous:** if there is no git project, choose **Harness preflight** and say so in the plan. Offer an optional "or name a repo to do a full Project install now" — do not require an answer to proceed with preflight after approval.

If the human names a clone URL, clone it to a durable path they approve (or `$HOME/projects/<slug>` / the harness computer's usual projects dir), then treat as **Project** or **Bind**.

### 0.1 Capability probes

Test **yourself**, not your product name. Record pass/fail for each:

| Id | Probe | Pass if |
| -- | ----- | ------- |
| **C1** | Spawn a trivial subagent ("reply with the single word pong"). Prefer a fresh-context option if the harness has one. | You get `pong` (or equivalent) back. Also record `context: fresh` or `context: inherited`. |
| **C2** | Start two independent subagents (or one background) without blocking the other | Both complete; or harness clearly supports parallel/background agents |
| **C3** | Shell + git: `git --version`. Isolation: git worktrees **or** disposable sandbox/computer checkouts (even if no project is open yet, confirm the mechanism exists) | Shell + git work; **and** at least one isolation mechanism is available (else fail C3 but note shell/git alone) |
| **C4** | List or invoke installed skills / open a known `SKILL.md` (plugin, skill store, or `.agents/skills/`) | You can invoke by name **or** read skill files on disk / in the harness skill store |
| **C5** | Ask the human one structured multiple-choice question or native approval card (or note that only plain chat exists) | Structured UI / approval card **or** plain chat only |
| **C6** | Which project instruction file does this harness load when a project is open? | One of: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, other (name it), or unknown. Bot personality files are not substitutes for the project factory block. |

If a probe is unsafe or impossible, mark **fail** and note why. Do not fake a pass.

### 0.2 Select mode

Using [factory/playbook/modes.md](factory/playbook/modes.md) rules:

- **Full** if C1 ∧ C2 ∧ C3
- **Serial subagents** if C1 and (not C2 or not C3)
- **Single-agent sequential** if not C1
- If C1 is pass but `context: inherited`, keep the mode; require ignore-prior-chat on every brief; downgrade Full → Serial if smokes show inherited noise causing mistakes

For **Harness preflight**, still record the provisional mode. Re-confirm C3 against the real repo during Bind.

### 0.3 Survey

**Always:**

- Whether harness factory home already exists (see H1): `$FACTORY_HOME` candidate paths
- Existing skills (plugin, skill store, `~/.agents/skills`, project `.agents/skills`)
- Whether `multi-agent-software-factory` (or older factory harness skill) is already in the skill store

**If Project or Bind:**

- `git remote -v`, default branch, cleanliness
- Instruction files: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, others
- `GLOSSARY.md`, `docs/adr/`, `.out-of-scope/`, `docs/agents/`, `docs/factory/`
- Check commands from manifests / CI
- Whether `docs/factory/VERSION` already exists (upgrade vs fresh)

### 0.4 Present the plan — GATE

Show the human:

1. **Scope:** Project | Harness preflight | Bind
2. Capability table (C1–C6) and **chosen mode**
3. What you will create/edit (project files and/or harness factory home / skill store)
4. Skills install route
5. Fresh vs upgrade vs bind

**Stop. Wait for explicit approval.** On rejection, adjust and re-ask. On approval, continue on the matching path below.

**Done-condition:** human approved the plan in this session.

---

# Path A — Project (full install)

Use when scope is **Project**.

## Step 1 — Install skills

Follow [factory/skills-manifest.md](factory/skills-manifest.md). Pick **exactly one** route family, in this order among those that work:

1. Managed plugin
2. Gemini CLI dual install
3. `npx skills@latest add mattpocock/skills` (include `setup-matt-pocock-skills`)
4. Project `.agents/skills/`
5. Harness skill store (and prefer a project `.agents/skills/` copy when IDEs share the repo)
6. Raw fetch

Also import [factory/harness-skill/SKILL.md](factory/harness-skill/SKILL.md) into the harness skill store when the store exists, named `multi-agent-software-factory`.

Never install both a managed plugin route and skills.sh.

**Done-condition:** required skills discoverable; harness skill present if a skill store exists; evidence = skill name list.

## Step 2 — Project setup

Run skill `setup-matt-pocock-skills` (or follow its `SKILL.md`).

Writes `docs/agents/issue-tracker.md`, `docs/agents/domain.md`, optional `triage-labels.md`, and an `## Agent skills` block into existing `CLAUDE.md` or `AGENTS.md` (ask which to create if neither exists). Do not create both.

**Done-condition:** those files exist; tracker answers a read call.

## Step 3 — Install factory payload into the project

### 3.1 Copy payload

| Source | Destination |
| ------ | ----------- |
| `factory/playbook/*` | `docs/factory/playbook/` |
| `factory/templates/*` | `docs/factory/templates/` |
| `factory/skills-manifest.md` | `docs/factory/skills-manifest.md` |
| `factory/harness-skill/SKILL.md` | `docs/factory/harness-skill/SKILL.md` |
| `factory/VERSION` | `docs/factory/VERSION` |

### 3.2 Write capabilities

Write `docs/factory/capabilities.md` using the template in [Capabilities template](#capabilities-template). Set `Scope: project` and `Project root:`.

### 3.3 Insert router block

1. Read [factory/AGENTS-block.md](factory/AGENTS-block.md) (markers inclusive).
2. Target = C6 file; if unknown, prefer existing `CLAUDE.md` else `AGENTS.md`; ask if neither. If one only `@`-includes the other, edit the file with real content.
3. Replace or append the marker block once.
4. Thin pointer file if another harness filename needs it. Never put the factory block in bot personality files.

### 3.4 Scaffold if missing

- `GLOSSARY.md` stub
- `docs/adr/.gitkeep`
- `.out-of-scope/README.md` one-liner

**Done-condition:** payload present; VERSION matches; one marker pair; re-run is a no-op.

## Step 4 — Brakes

1. Propose typecheck / lint / test commands from the survey.
2. Run each (deps only if approved in the plan).
3. Write `docs/factory/checks.md` with the command table.
4. Offer `CODING_STANDARDS.md` stub only if the human says yes.
5. If no CI/pre-commit guardrail, note `No guardrail detected — first retro item.`

**Done-condition:** `checks.md` exists; each command exercised once with recorded outcome.

## Step 5 — Smoke tests

Dry probes; prefer no product-code edits.

1. **Question:** real repo question; cite files; no product edits.
2. **Bug dry-run:** Triager brief outline only; no code edits.
3. **Feature dry-run:** stop at human gate; no implement.
4. **Mode:** C1 still works; inherited-context brief obeyed if needed; Full confirms isolation.

**Done-condition:** applicable smokes pass, or one fix then stop with failures listed.

## Step 6 — Report

Tell the human: mode, what changed, how to use Question/Bug/Feature, what was skipped, first action = run a real Question flow.

**Done-condition:** report delivered.

---

# Path B — Harness preflight (no project yet)

Use when scope is **Harness preflight**. Goal: the next time the human points at a repo, Bind is fast and does not re-ask skill basics.

## H1 — Resolve harness factory home

Pick the first writable durable directory (create if needed after approval):

1. `$FACTORY_HOME` if already set
2. `$HOME/factory` (or the bot/computer home equivalent, e.g. the agent's home directory + `/factory`)
3. OS temp only as last resort (warn that it may not survive)

Record the chosen path as **harness factory home**.

## H2 — Install skills into the harness

Follow [factory/skills-manifest.md](factory/skills-manifest.md) with preference order flipped for no-project:

1. Harness skill store (best when no project tree)
2. Managed plugin / Gemini / skills.sh if those CLIs work in this environment
3. Raw fetch into whatever global skills directory the harness already reads
4. Do **not** invent a fake project just to fill `.agents/skills/`

Also import [factory/harness-skill/SKILL.md](factory/harness-skill/SKILL.md) as skill `multi-agent-software-factory`.

**Done-condition:** required Matt Pocock skills discoverable **and** `multi-agent-software-factory` is invokable or readable in the skill store / skills dir.

## H3 — Cache the factory payload at harness home

Copy from the resolved `factory/` source:

| Source | Destination under harness factory home |
| ------ | -------------------------------------- |
| `playbook/*` | `playbook/` |
| `templates/*` | `templates/` |
| `skills-manifest.md` | `skills-manifest.md` |
| `harness-skill/SKILL.md` | `harness-skill/SKILL.md` |
| `AGENTS-block.md` | `AGENTS-block.md` |
| `VERSION` | `VERSION` |
| this SETUP.md (optional but useful) | `SETUP.md` |

Write `capabilities.md` using the [Capabilities template](#capabilities-template) with `Scope: harness-preflight`, `Project root: (none)`, and `Harness factory home:`.

Write `BIND.md` with exactly:

```markdown
# Bind the software factory to a project

Harness preflight is done. When you have a git working tree:

1. Open or clone the project.
2. Tell the agent: "Bind the multi-agent software factory to this project"
   (or re-run SETUP.md — it will choose Bind).
3. Approve the Bind plan. The agent runs project setup, copies docs/factory/,
   inserts the router block, records checks, and smokes.
```

**Done-condition:** harness home contains `VERSION`, `playbook/`, `templates/`, `capabilities.md`, `BIND.md`, and `harness-skill/SKILL.md`.

## H4 — Harness smokes

1. Invoke or open `multi-agent-software-factory` and confirm it describes Question/Bug/Feature routing.
2. Confirm at least `ask-matt` or `grill-with-docs` is reachable.
3. Re-run C1 ping if mode is Full or Serial.

Skip project question/bug/feature smokes (no repo).

**Done-condition:** harness smokes pass.

## H5 — Report

Tell the human:

1. Scope was **harness preflight** (no project was open)
2. Mode (provisional) and capabilities
3. Where the harness factory home is
4. Skills are ready in the harness
5. **What to say next:** point at a repo (path or clone URL), then "bind the multi-agent software factory to this project"
6. Optional: they may name a repo now to Bind in this same session

**Done-condition:** report delivered. Do not leave the human stuck choosing among coding teammates unless they ask.

---

# Path C — Bind to a project

Use when scope is **Bind** (preflight already done, project now available).

1. Confirm harness factory home and that required skills still resolve. If skills are missing, run Path A Step 1 (or H2) first.
2. Run Path A **Steps 2–6** into the project.
3. When copying the payload (Step 3), prefer the **harness factory home** cache if its `VERSION` matches the SETUP source; otherwise fetch from the SETUP payload location.
4. Copy harness `capabilities.md` into the project, then update it: `Scope: project`, fill `Project root`, re-check C3 against this repo.
5. Report that Bind completed and the three entry points are live.

**Done-condition:** same as Path A Steps 2–6.

---

## Capabilities template

```markdown
# Factory capabilities

- Detected: <ISO date>
- Scope: project | harness-preflight
- Mode: Full | Serial subagents | Single-agent sequential
- C1 (subagent): pass | fail — context: fresh | inherited — <note>
- C2 (parallel/background): pass | fail — <note>
- C3 (shell/git + isolation): pass | fail — isolation: worktree | sandbox-checkout | none — <note>
- C4 (skills by name): pass | fail — <note>
- C5 (structured questions / approvals): pass | fail — <note>
- C6 (instruction file): <filename or unknown>
- Project root: <absolute path or (none)>
- Harness factory home: <absolute path or (none)>
- Notes directory (exploration): <absolute path>
- Skills install route used: <plugin | gemini | skills.sh | project-agents-skills | harness-skill-store | raw>
```

---

## Upgrade

Re-run this file from the top.

1. Detect scope again (project vs harness vs bind).
2. Compare source `factory/VERSION` to `docs/factory/VERSION` and/or harness home `VERSION`.
3. Refresh payload files and the marker block; refresh the harness skill store entry for `multi-agent-software-factory`.
4. Leave unrelated user edits untouched.
5. Re-run brakes/smokes appropriate to the scope.

---

## Uninstall

Only when the human explicitly asks:

**Project:** remove the marker block and `docs/factory/`. Leave skills, glossary, ADRs, `docs/agents/` unless asked.

**Harness:** remove the harness factory home directory and the `multi-agent-software-factory` skill-store entry. Leave Matt Pocock skills unless asked to remove those too.

Report what was removed and what was left.

---

## Quick reference

| Scope | Steps |
| ----- | ----- |
| Project | 0 → 1 → 2 → 3 → 4 → 5 → 6 |
| Harness preflight | 0 → H1 → H2 → H3 → H4 → H5 |
| Bind | 0 → (skills if needed) → 2 → 3 → 4 → 5 → 6 |
