# Knowledge-base vault setup

I want you to bootstrap a personal knowledge-base vault for me. This is an Obsidian-compatible vault that is also designed for you to operate as a writing/curating partner.

## End state

Land everything at `~/obsidian-vault/`. The shape:

```
~/obsidian-vault/
├── SCHEMA.md              ← the constitution; what goes where, what is refused
├── tasks-inbox.md         ← the only place new tasks are appended
├── tasks-dashboard.md     ← Obsidian Tasks plugin query view (read-only)
├── wiki/
│   ├── index.md           ← catalogue of every wiki page; updated on every wiki write
│   ├── concepts/
│   ├── projects/
│   └── people/
├── raw/
│   ├── meetings/
│   ├── decisions/
│   └── research/
└── draft/                 ← my staging area; you ignore it entirely
```

## The philosophy that makes this work

Three lanes, strictly separated:

- **`draft/`** is mine. Verbatim staging. You never read, refine, or move anything here. It is where I dump fragments before deciding what they are.
- **`raw/<subfolder>/`** is the shared input layer. I drop captured sources (meeting notes, transcripts, correspondence). You file operational artefacts you produce (research reports, briefings, captures). Nothing in `raw/` ever moves after creation.
- **`wiki/`** is yours to write, mine to read. **Compiled** knowledge only — concepts, projects, people. Not operational outputs. Pages cite their `raw/` sources via frontmatter; corrections flow `draft/` → `raw/` → re-ingest, never quiet wiki edits.

Non-negotiable:

- **`tasks-inbox.md` is the only task surface.** Never create new task files. Appends only, in the Obsidian Tasks plugin format: `- [ ] description 📅 YYYY-MM-DD ⏫ ➕ YYYY-MM-DD` (priority: ⏫ high · 🔼 medium · 🔽 low).

## What each file should say

Write proper, terse, lived-in content — not bullet-list dumps. Use British English in prose (colour, organisation). Use American English in code identifiers if any appear. Default voice: confident, direct, no padding.

- **`SCHEMA.md`** — the constitution. Versioned (`Version: 1`, `Last updated: <today>`). Lays out: vault structure tree, lane boundaries (`draft`/`raw`/`wiki`), what each lane refuses, the two non-negotiables above, frontmatter convention (wiki pages carry `sources:` and `updated:`; raw has no mandate), wikilink format (`[[wiki/concepts/foo|Display Name]]`, first mention only, never inside markdown tables), and the task format. Keep it readable.
- **`tasks-inbox.md`** — header plus a couple of sensible H2 sections (e.g. `## Personal`, `## Work`). One placeholder task per section so the format is visible.
- **`tasks-dashboard.md`** — an Obsidian Tasks plugin dashboard with a few useful queries (open tasks, due this week, overdue, by priority). Use the plugin's `tasks` code-fence syntax.
- **`wiki/index.md`** — a catalogue header plus an empty table with columns `Page | Topic | Updated`. No example rows.
- **`raw/`, `draft/`, and the `wiki/` subfolders** — create the directories. No example markdown.

## The four skills

Create four project-scoped skills to work with the vault. Use the /skill-creator skill to do this.

- **`vault-file`** — the chokepoint for every new file landing in `draft/` or `raw/<subfolder>/`. Procedure: confirm SCHEMA preconditions (target subfolder exists if `raw/`; refuse `wiki/` and `tasks-inbox.md`); write the file; ask if user wants to ingest.
- **`vault-ingest`** — turn one or more `raw/` sources into compiled `wiki/` pages. Procedure: read the source; consult existing wiki pages via grep; write/update the wiki page(s) in my voice (sources cited, not pasted; wikilinks first-mention only); update `wiki/index.md`; touch related pages with `> ⚠️ Updated YYYY-MM-DD — <what changed>` if they need it; append to `tasks-inbox.md` if the source implies actions.
- **`vault-query`** — answer a question from the wiki. Procedure: read `wiki/index.md` first (it exists precisely so you don't blind-grep); pick one or two candidate pages; read them in full; follow wikilinks as needed; only fall back to `raw/` if the wiki page is thin or stale; synthesise the answer in British English with citations like `(source: wiki/projects/foo.md)`; never fabricate — if the wiki doesn't know, say so; flag gaps and offer to ingest.
- **`vault-lint`** — periodic hygiene. Six classes: broken wikilinks (**fix in place**), orphan pages, stale claims (wiki `updated:` predates a newer raw source), contradictions, missing pages (a name referenced ≥3× across wiki without its own page), and folder structure imbalances. Only broken links are auto-fixed; everything else is *proposed* and waits for my greenlight before any edit. Append a `[YYYY-MM-DD] lint | Pass #n` entry with counts.

## Vault linting cron job

Add a new cron job to call `vault-lint` skill every Sunday morning 5:12AM. Register it in `cron-prompts.md` so it survives session restarts.

## Update your CLAUDE.md

Update your `CLAUDE.md` to:

- surface the existence of the vault
- points at `SCHEMA.md` as the source of truth for the vault
- always file knowledge or quick notes to the vault using `vault-file` skill
- queries information from the vault using `vault-query` skill

## What I do not want

- Do not install anything (Obsidian, plugins, etc.) — just write content compatible with them.
- Do not invent example data inside `wiki/` — leave it empty and ready, with only `wiki/index.md` and the directory placeholders.

## When you finish

Reply with: the absolute path of the vault, the count of files written, and a one-line summary of what's ready to use. Then stop — no further suggestions until I drive.
