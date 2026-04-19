# everywhere

A private GitHub repo of plain Markdown for personal task management — GTD-style, but the agent handles the clarify and organize steps. You capture; any coding agent reads this file and runs the system. Git is your undo.

Save this as `AGENTS.md` at the repo root — the open standard any coding agent reads (Codex, Claude Code, Cursor, OpenCode, Aider, Jules, local models on Ollama, …). Run from your phone via **Claude Code Web** or **Codex Web**; from desktop via any CLI. Attach the repo, say *"process inbox"*, it works across any device, model, or provider. No app, no database, no lock-in — just files, git, and grep.

---

## The one idea

**A task file is a durable thread.** One outcome = one file in `/tasks/`. It survives context resets, agent swaps, laptop closes, device switches. Everything else follows.

---

## Layout

```
/inbox/       raw captures — text, voice transcripts, PDFs, images, dumps
/tasks.md     flat dashboard — Next / Waiting / Done (GTD lists)
/tasks/       durable thread files for complex or long-running outcomes
/projects/    optional — ongoing domains with their own context
/wiki/        LLM-compiled notes, one concept per file
/sources/     optional — preserved raw source material (papers, clippings)
/scripts/     tiny helpers: rebuild-status.sh, lint.sh
/status.md    auto-generated snapshot for mobile review
/AGENTS.md    this file
```

A **slug** is a short, URL-safe filename — `renew-passport`, not `Renew Passport!`. Use kebab-case. Dated slugs like `2026-04-18-renew-passport` sort naturally.

`status.md` and `tasks.md` are committed so you can scan repo state on a phone without running anything. Binaries (PDFs, images, audio) live in `/inbox/`, `/sources/`, or project folders; agents summarize them to Markdown and link back.

**Bootstrap:** on an empty repo, create paths lazily as you first write into them — no upfront scaffolding, no `.gitkeep` files.

---

## Task vs project

- **Simple task** → a line in `tasks.md`. "Buy milk", "pay invoice." One step, done today.
- **Durable task** (`tasks/<slug>.md`) → one outcome, multi-step, spans sessions. "Renew passport", "migrate DNS." Done when the outcome is reached.
- **Project** (`projects/<slug>/`) → an ongoing domain that *spawns* many tasks and has its own notes, drafts, or assets. "SaaS launch", "PhD thesis." Never "done" in one step.

Rule of thumb: if it has sub-notes, drafts, or keeps generating new tasks, it's a project. Otherwise it's a task.

---

## File shapes

Front-matter is **grep bait**: few keys, enum-like values. Add keys (`priority`, `assignee`, `tags`) as needed.

**Flat task line** in `tasks.md`:
```md
- [ ] text (added YYYY-MM-DD)
- [ ] text — @project-slug (added YYYY-MM-DD, due YYYY-MM-DD)
- [ ] text — waiting on: reason (since YYYY-MM-DD)
- [x] text (done YYYY-MM-DD)
```

**Durable task file** — `tasks/<slug>.md`
```md
---
status: open        # open | doing | blocked | done | dropped
project: alpha      # optional
created: 2026-04-18
due: 2026-05-01
---
# Renew passport before Italy trip
## Context
## Next actions
- [ ] dig out old passport
## Log
```

**Project** — `projects/<slug>/README.md`: goal, current state, one clear next action, links. Drop `notes.md`, drafts, and assets in the folder.

**Wiki note** — `wiki/<slug>.md`: one idea per file. `sources:` accepts `/inbox/` paths, URLs, `/sources/` paths, or free text. Every claim traces to a source.

---

## Grep-linking

Link any file to any other by writing its **relative path** inline. `rg 'tasks/renew-passport'` finds every mention across tasks, wiki, projects — no special syntax, just paths in prose. `@project-slug` in `tasks.md` lines is the convention for task-to-project links. Structured fields (`project:`, `sources:`) are for things you'll filter on; everything else is prose references that grep the same.

---

## Flow (GTD, but mostly automated)

1. **Capture** anywhere → commit to `/inbox/`. Don't classify.
2. **Clarify + organize** — say *"process inbox"*. Agent classifies each item (see procedure), updates `tasks.md`, commits.
3. **Reflect** — say *"weekly review"*. Agent sweeps open tasks, stale projects, overdue items; commits.
4. **Engage** — say *"next"* or `rg '^- \[ \]' tasks.md`. Pick one and do it.
5. **Undo** — anything wrong? Say *"revert the last change"* or *"unmark that task"*. Git history is the undo stack.

---

## Inbox classification procedure

For each file in `/inbox/`:

1. **Simple actionable item?** → append a line to `tasks.md` under Next.
2. **Complex outcome needing its own thread?** → create `tasks/<slug>.md`.
3. **Belongs to an existing project?** → add to `projects/<slug>/` or update its README.
4. **Fact, concept, or reference worth preserving?** → create/update a file in `wiki/`.
5. **Raw source worth preserving verbatim?** → move to `sources/`, summarize into a wiki note that links back.
6. **Cannot classify with confidence?** → leave in place, add a `> TODO: classify` line. Do not guess.

After classification, **delete** moved inbox items. Unclassified ones stay.

---

## Cross-device

- **Capture** from anywhere — desktop editor, phone Shortcut → GitHub Contents API, browser extension, email-to-inbox, `gh api` from any terminal. Or open a GitHub issue; agents treat issues as inbox items.
- **Run an agent from your phone** via **Claude Code on the web** (claude.ai/code, iOS app) or **Codex Web** — attach the repo, type a prompt, the agent runs on a cloud VM and pushes commits. Desktop users: any CLI harness (Claude Code, Codex, Aider, OpenCode) or a self-hosted model + git-capable harness.
- **See what happened** via GitHub app, web, or `gh log`. Parallel runs on multiple devices are fine — they stack.

---

## Working rules

### Grep first, read second

YAML front-matter + `rg` is the index.

```bash
rg '^status: open$' tasks/ -l       # every open durable task
rg '^- \[ \]' tasks.md              # every open simple task
rg '^project: alpha$' tasks/ -l     # tasks in project alpha (or @alpha in tasks.md)
rg -l 'tasks/passport' .            # everything linking to a task
ls inbox/                           # pending captures
git log --oneline -20               # recent changes (undo targets)
```

### Git flow

- **Pull first.** Every run starts with `git pull --rebase`. Catches other devices' work; rebases cleanly onto it.
- **One logical change per commit.** Message: `<verb>: <what>` — `process: 4 inbox items`, `done: renew-passport`. Commit to main directly, or push a branch + auto-merge PR if your harness requires it (Claude Code Web, Codex Web). Same outcome.
- **Undo is `git revert`.** "Unmark that task" → agent edits the file back and commits. "Revert yesterday's changes" → `git revert <sha>`. Rewinding further → `git reset --hard <sha>` with user confirmation.
- **If uncertain, don't commit.** Leave the repo untouched, report the ambiguity in your reply. Never invent facts or file paths.

### Workflow

- **Run `scripts/rebuild-status.sh` if present** before committing; same for `scripts/lint.sh`. Otherwise regenerate `status.md` inline.
- **Wiki notes: rewrite in place.** Task logs, `tasks.md` Done section, and project drafts: append.
- **On rename or merge of a file,** grep for the old path and update every reference.

---

## Not

Not a build system. Not a vector DB. Not app-coupled. Not single-agent, not single-provider, not frontier-only. Anything that reads this file and commits to git can contribute.

---

## Why it holds up

Grep is the API. Git history is the undo stack. Markdown + git is the substrate. This file is the contract. Everything else is convention, and conventions are cheap to change.
