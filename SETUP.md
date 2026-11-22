# SETUP.md — Multi-Agent Software Factory

**You are an agent.** Execute this file to install the factory into the **current project**: the git working tree you can write (IDE workspace, checkout on a computer/sandbox, or clone the human pointed you at). Do not invent steps. Do not skip done-conditions.

## Contract

1. **Read this entire file first**, then explore the project read-only, then present a plan, then wait for approval before writing.
2. **Ask before writing.** Step 0 ends at a human gate. No installs or file edits before approval.
3. **Never overwrite user content** outside the factory markers and `docs/factory/`. If a target file exists with custom content, update in place or ask.
4. **Every write is idempotent.** A second run must produce no meaningful diff once installed at the same `VERSION`.
5. **Stop and report** on any failed done-condition after at most one fix attempt for that step.
6. **Payload location.** The installable payload lives next to this file under `factory/`. Resolve it as:
   - If this `SETUP.md` was opened by path → use `<directory-containing-SETUP.md>/factory/`.
   - If this `SETUP.md` was fetched by URL → fetch sibling paths under the same base URL (`factory/VERSION`, `factory/playbook/*`, etc.).
7. **Skill invocation fallback** (use this wording everywhere): when a procedure says "run skill X": (1) if the harness can invoke a skill by name, do that; (2) otherwise locate and read that skill's `SKILL.md` and follow it. Never invent a substitute when the skill file is reachable.

---

## Step 0 — Detect (read-only)

### 0.1 Capability probes

Test **yourself**, not your product name. Record pass/fail for each:

| Id | Probe | Pass if |
| -- | ----- | ------- |
| **C1** | Spawn a trivial subagent ("reply with the single word pong"). Prefer a fresh-context option if the harness has one. | You get `pong` (or equivalent) back. Also record `context: fresh` or `context: inherited` (inherited = child can see parent chat). |
| **C2** | Start two independent subagents (or one background) without blocking the other | Both complete; or harness clearly supports parallel/background agents |
| **C3** | Shell + git: `git --version`. Isolation: either `git worktree list` / create a worktree, **or** confirm you can open a disposable sandbox/computer with its own checkout of this repo | Shell + git work; **and** at least one isolation mechanism is available (else fail C3 but note shell/git alone) |
| **C4** | List or invoke installed skills / open a known `SKILL.md` (plugin, skill store, or `.agents/skills/`) | You can invoke by name **or** read skill files on disk / in the harness skill store |
| **C5** | Ask the human one structured multiple-choice question or native approval card (or note that only plain chat exists) | Structured UI / approval card **or** plain chat only |
| **C6** | Which project instruction file does this harness load? | One of: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, other (name it), or unknown. Bot personality files (e.g. `SOUL.md`) are not substitutes for the project factory block. |

If a probe is unsafe or impossible, mark **fail** and note why. Do not fake a pass.

### 0.2 Select mode

Using [factory/playbook/modes.md](factory/playbook/modes.md) rules:

- **Full** if C1 ∧ C2 ∧ C3 (isolated workspaces: worktrees or sandbox checkouts)
- **Serial subagents** if C1 and (not C2 or not C3)
- **Single-agent sequential** if not C1
- If C1 is pass but `context: inherited`, keep the mode from the rules above; require ignore-prior-chat on every brief; downgrade Full → Serial if smokes show inherited noise causing mistakes

### 0.3 Project survey

Read whatever exists; do not assume:

- `git remote -v`, default branch, whether the tree is clean
- Existing instruction files: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, others
- Existing skills install (plugin, `~/.agents/skills`, project `.agents/skills`, etc.)
- `GLOSSARY.md`, `docs/adr/`, `.out-of-scope/`, `docs/agents/`, `docs/factory/`
- Check commands from `package.json` / `Makefile` / `pyproject.toml` / CI workflows (typecheck, lint, test)
- Whether `docs/factory/VERSION` already exists (upgrade vs fresh)

### 0.4 Present the plan — GATE

Show the human:

1. Capability table (C1–C6) and **chosen mode**
2. Files you will **create**
3. Files you will **edit** (especially the instruction file for C6)
4. Skills install route you intend (plugin, Gemini, skills.sh, project `.agents/skills/`, harness skill store, or raw)
5. Whether this is a **fresh install** or **upgrade** from `docs/factory/VERSION`

**Stop. Wait for explicit approval.** On rejection, adjust the plan and re-ask. On approval, continue.

**Done-condition:** human approved the plan in this session.

---

## Step 1 — Install skills

Follow [factory/skills-manifest.md](factory/skills-manifest.md). Pick **exactly one** route family, in this order of preference among those that actually work in this harness:

1. Managed plugin (Claude Code / Codex / Copilot) if clearly supported
2. Gemini CLI dual `skills install` if that CLI is available
3. `npx skills@latest add mattpocock/skills` (omit `-a` to choose; include `setup-matt-pocock-skills`)
4. Copy required skills into the project's `.agents/skills/` (preferred when CLIs are missing but the project tree is writable, e.g. inside a sandbox/computer)
5. Import into the harness account/space skill store (and still prefer a project `.agents/skills/` copy when IDEs will share the repo)
6. Raw fetch of each `SKILL.md` into the skills folder the harness already reads

Never install both a managed plugin route and skills.sh.

**Done-condition:**

- `ask-matt` and `setup-matt-pocock-skills` are discoverable **or** their `SKILL.md` files are readable on disk
- Every skill listed in the manifest tables is present
- Evidence: list the skill names you verified

---

## Step 2 — Project setup (tracker, labels, domain docs)

Run skill `setup-matt-pocock-skills` (or read and follow its `SKILL.md`).

It will explore, ask the human (issue tracker, triage labels if triage is installed, domain doc layout), and write:

- `docs/agents/issue-tracker.md`
- `docs/agents/domain.md`
- `docs/agents/triage-labels.md` (if triage installed)
- An `## Agent skills` block into the existing `CLAUDE.md` or `AGENTS.md` (it will ask which to create if neither exists)

Do not re-implement that skill. Do not create both `CLAUDE.md` and `AGENTS.md`.

**Done-condition:**

- `docs/agents/issue-tracker.md` and `docs/agents/domain.md` exist
- Tracker answers a read call appropriate to the choice (e.g. `gh repo view`, `glab repo view`, or a readable `.scratch/` layout)
- Evidence: cite the files and the read-call output summary

---

## Step 3 — Install factory payload

### 3.1 Copy payload

From the resolved `factory/` directory (see Contract §6), install into the project:

| Source | Destination |
| ------ | ----------- |
| `factory/playbook/*` | `docs/factory/playbook/` |
| `factory/templates/*` | `docs/factory/templates/` |
| `factory/skills-manifest.md` | `docs/factory/skills-manifest.md` |
| `factory/VERSION` | `docs/factory/VERSION` |

Replace files under `docs/factory/playbook/` and `docs/factory/templates/` on upgrade. Do not delete user-created files elsewhere under `docs/factory/` except when replacing known payload filenames.

### 3.2 Write capabilities

Write `docs/factory/capabilities.md`:

```markdown
# Factory capabilities

- Detected: <ISO date>
- Mode: Full | Serial subagents | Single-agent sequential
- C1 (subagent): pass | fail — context: fresh | inherited — <note>
- C2 (parallel/background): pass | fail — <note>
- C3 (shell/git + isolation): pass | fail — isolation: worktree | sandbox-checkout | none — <note>
- C4 (skills by name): pass | fail — <note>
- C5 (structured questions / approvals): pass | fail — <note>
- C6 (instruction file): <filename or unknown>
- Project root: <absolute path of the git working tree being instrumented>
- Notes directory (exploration): <absolute path outside repo, e.g. $TMPDIR/factory-notes/<project> >
- Skills install route used: <plugin | gemini | skills.sh | project-agents-skills | harness-skill-store | raw>
```

Create the notes directory if missing. If the harness has no durable path outside the repo, put notes under `$TMPDIR` on the computer/sandbox and record that path.

### 3.3 Insert router block

1. Read [factory/AGENTS-block.md](factory/AGENTS-block.md) (content between markers inclusive).
2. Target file = C6's instruction file. If C6 is unknown: prefer existing `CLAUDE.md` else existing `AGENTS.md`; if neither exists, **ask** which to create (same rule as setup-matt-pocock-skills). If both exist and one only `@`-includes the other, edit the file that holds the real content.
3. If markers `<!-- factory:start -->` … `<!-- factory:end -->` already exist → replace the block in place.
4. Else append the block once at the end of the file (with a blank line before it).
5. If the harness reads a *different* filename than the one you edited, create that other file as a **thin pointer** only:

   ```markdown
   See AGENTS.md (or CLAUDE.md) for project agent instructions, including the multi-agent software factory block.
   ```

   Do not duplicate the full factory block into two full copies unless the human asks. Do not put the factory block into bot personality files; keep it in the project instruction file.

### 3.4 Scaffold durable dirs if missing

Create empty scaffolds only when absent:

- `GLOSSARY.md` — stub with `# Glossary` and one line: `Terms agreed during grilling live here.`
- `docs/adr/` — add `.gitkeep` if the directory would otherwise be empty
- `.out-of-scope/` — add a one-line `README.md`: `Rejected enhancements live here as short markdown files.`

**Done-condition:**

- All payload paths exist
- `docs/factory/VERSION` matches source `factory/VERSION`
- Instruction file contains exactly one factory marker pair
- Re-running step 3 would produce no content diff
- Evidence: `find docs/factory -type f | sort` (or equivalent)

---

## Step 4 — Brakes (checks and standards)

1. From the project survey, propose typecheck / lint / test commands.
2. Confirm each proposed command **runs** (exit code meaningful; install deps only if the human approved dependency installs in the plan).
3. Write `docs/factory/checks.md`:

   ```markdown
   # Checks (Implementer hard gate)

   All of these must pass before an Implementer reports done.

   | Name | Command |
   | ---- | ------- |
   | … | … |

   ## Notes

   - …
   ```

4. If `CODING_STANDARDS.md` (or the project's named standards file) is missing, **offer** a short stub for judgement-call standards the Spec/Standards reviewers can use. Create it only if the human says yes.
5. If the repo has **no** guardrail (no CI job and no pre-commit running lint/typecheck/test), do **not** invent a full CI system. Record in `docs/factory/checks.md` under Notes: `No guardrail detected — first retro item.`

**Done-condition:** `docs/factory/checks.md` exists; each listed command was executed once with a recorded outcome (pass, fail, or skipped-with-reason).

---

## Step 5 — Smoke tests

Run these as **dry probes**. Prefer not to leave product-code edits. If a probe would edit code, stop before writing and still report whether the *decision path* stopped at the right gate.

### 5.1 Question smoke

- Ask yourself a real question about this repo (e.g. "What is the project's entrypoint?").
- Follow `docs/factory/playbook/flow-question.md` briefly (at least one Explorer pass or equivalent phase).
- **Pass if:** the answer cites concrete files; no product files changed by the factory.

### 5.2 Bug smoke (dry-run)

- Invent or use a described bug ("button X does nothing") without fixing it.
- Follow Triager steps only: gather context, recommend category/state, sketch what an Agent Brief would contain.
- **Pass if:** you produce a brief outline and a role recommendation **without** editing product code.

### 5.3 Feature smoke (dry-run)

- Follow only grilling kickoff + size decision from `flow-feature.md` on a tiny hypothetical feature.
- **Pass if:** you stop at a human gate (grilling or size/tickets) and do not start implementing.

### 5.4 Mode probe

- If mode is Full or Serial: confirm C1 still works with a one-line subagent report. If `context: inherited`, confirm the child followed a brief that said to ignore prior chat.
- If mode is Full: confirm the recorded isolation mechanism (worktree or sandbox checkout) is still available.
- If Single-agent: confirm you documented the cold-reviewer rule in the smoke report.

**Done-condition:** all applicable smokes pass, or one fix attempt then stop with failures listed.

---

## Step 6 — Report

Tell the human:

1. **Mode** in effect and capability summary
2. **What changed** (skills route, files created/edited)
3. **How to use** the three entry points in plain language:
   - Question: ask about the project
   - Bug: report something broken
   - Feature: ask for something new (expect grilling)
4. **What was skipped** and why
5. **First recommended action:** run a real Question flow on this repo

**Done-condition:** report delivered in the chat.

---

## Upgrade

Re-run this file from the top.

1. Compare `factory/VERSION` (source) to `docs/factory/VERSION` (installed).
2. Re-detect capabilities (harness may have changed).
3. Replace payload files under `docs/factory/playbook/`, `templates/`, `skills-manifest.md`, and the marker-delimited block.
4. Leave user edits outside markers and outside replaced payload filenames untouched.
5. Re-run brakes if check commands may have changed; re-run smokes.

---

## Uninstall

Only when the human explicitly asks to uninstall:

1. Remove the `<!-- factory:start -->` … `<!-- factory:end -->` block from the instruction file(s).
2. Remove `docs/factory/` (entire tree).
3. **Leave in place** unless the human asks otherwise: installed skills, `GLOSSARY.md`, `docs/adr/`, `.out-of-scope/`, `docs/agents/`, `CODING_STANDARDS.md`, tracker issues, git branches/PRs.

Report what was removed and what was left.

---

## Quick reference — steps

| Step | Name | Gate / stop |
| ---- | ---- | ----------- |
| 0 | Detect + plan | Human approves before writes |
| 1 | Install skills | Discoverability of required skills |
| 2 | setup-matt-pocock-skills | `docs/agents/*` + tracker read |
| 3 | Payload + router block | Idempotent `docs/factory/` + markers |
| 4 | Brakes | `checks.md` with exercised commands |
| 5 | Smokes | Question / bug dry / feature dry / mode |
| 6 | Report | Usage + first action |
