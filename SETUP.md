# AI Assistant - Autonomous Setup Guide

You are Claude Code. The user wants you to set up a 24/7 personal AI assistant system inside the current project directory. Each path in this guide is relative to the project root. Follow every step below in order. Do NOT skip steps.

---

## STEP 1: Check the system readiness

Run these commands to verify the system is ready:

```bash
node --version
python3 --version
git --version
which curl jq
```

If anything is missing, install it before continuing.

---

## STEP 2: Backend API Server

This is a lightweight Flask + SQLite server that stores cron jobs status and for future capability expansion.

```bash
mkdir -p ./backend
cd backend
python3 -m venv venv
source venv/bin/activate
pip install flask sqlite-utils
```

Have Claude Code generate `./backend/api_server.py` as the core server in Python (Flask + SQLite) with these endpoints:

**Crons snapshot:**

| Method | Endpoint      | Purpose                                                                                                  |
| ------ | ------------- | -------------------------------------------------------------------------------------------------------- |
| POST   | /cron/active  | Replace the snapshot of currently scheduled crons `{crons:[{job_id, label, cron_expr, prompt_preview}]}` |
| GET    | /cron/active  | Runtime cron list                                                                                        |
| GET    | /cron/prompts | Parse `./cron-prompts.md` into sections                                                                  |

The server should:

- Listen on `0.0.0.0:7777`
- Use SQLite via `sqlite-utils`
- Store the database at `./backend/state.db`
- Enable CORS for all origins
- Serve a web visualisation at `/ui`

### Web Visualisation

Also create an `index.html` in the same directory — a single-page web app served at `/ui` with one tab:

1. **Logs** — Chronological view of all conversation logs with collapsible date groups and search.
2. **Crons** — two-column diff: runtime-active with live countdowns to next fire (updated every second via JS cron-next), and persisted prompts from `./cron-prompts.md` with sync badges (`synchronised` / `⚠ not created`).

A CSS variable `--topbar-h` is kept in sync with the actual topbar height so panels never overlap when the topbar wraps on mobile.

The server serves this file when a browser hits `GET /ui`.

Start it:

```bash
NOVA_DB_PATH=./backend/state.db nohup python3 ./backend/api_server.py > ./backend/logs/api_server.log 2>&1 &
```

Verify:

```bash
curl -s http://127.0.0.1:7777/health
# Should return: {"status":"ok"}
# Open http://localhost:7777/ui in browser to see the visualisation
```

---

## STEP 3: Create CLAUDE.md

This is the brain of the system. Write this to `./CLAUDE.md`.

But before writing the file, ask the user two questions back-to-back:

> "What name should I respond to? (e.g. Nova)"

> "What personality should I have? (e.g. 'dry, sarcastic wit — quip whenever there's a chance', or 'warm, concise, no fluff')"

Capture both answers verbatim — the personality description is the user's voice direction and should not be paraphrased. Use the name as the title of the document (`# <NAME>`) and fold the personality description into a `## Personality` section immediately under it. The name and personality are load-bearing: every Telegram reply, welcome message, and briefing should reflect them.

Next, determine the user's timezone by asking the following question:

> "Where are you based in? This will decide my reporting timezone (e.g. 'Kuala Lumpur', 'Jakarta')"

Determine the real IANA zone from the user's answer. Validate that the value is a real IANA zone before accepting it — `TZ="$USER_IANA_TIMEZONE" date` must succeed without a "cannot access" warning.

~~~markdown

# <NAME>

You are <NAME>, a personal AI assistant. You communicate via Telegram.

## Personality

<PERSONALITY>

Stay in character in every message — Telegram replies, daily briefings, cron notifications, welcome messages, proposals.

## Timezone

- **User timezone**: <USER_IANA_TIMEZONE>. Always use this for every human-facing time: Telegram replies, daily briefings, web UI and local-time targets for every scheduled cron.

## Language

British English in prose and UI text; American English in code identifiers.

## Session startup

At the start of each session, perform these steps automatically:

1. **Check backend API health**: `curl -s http://127.0.0.1:7777/health` — if the API is down, start it automatically using `NOVA_DB_PATH=./backend/state.db nohup python3 ./backend/api_server.py > ./backend/logs/api_server.log 2>&1 &` and wait 2 seconds before verifying again.
2. **Create all cron jobs** (see `./cron-prompts.md`). The heartbeat and briefing crons verify that all jobs are active and recreate any that are missing.
3. **Sync the runtime cron snapshot**: `POST /cron/active` with the current job list so the Crons dashboard tab can show live countdowns.

~~~

---

## STEP 4: Create the cron jobs

| # | Name | Schedule | What it does |
|---|------|----------|-------------|
| 1 | Cron watchdog | Every 6h (:23) | Verify no crons are about to expire (7-day TTL), alert if any are |
| 2 | Heartbeat | Every 1h (:43) | System state check, verify all 4 crons active, recreate missing. Social message 1x/day (50/50) if no alerts. Silent between 00:00-08:00 |
| 3 | Memory API health | Every 3h (:33) | Check API health, auto-restart if down, notify user of failures |
| 4 | Daily briefing | Daily ~9:00 AM | Weather, currencies, news, movies. |

The scheduled time in the table above is expressed in the user timezone. Adjust to the server time accordingly (likely UTC).
Persist these 4 prompts to `./cron-prompts.md` so they survive session restarts. On each new session, the startup steps recreate the cron jobs from this file.

---

## STEP 5: Verify everything

Run these checks:

```bash
# Backend API running?
curl -s http://127.0.0.1:7777/health

# CLAUDE.md exists?
head -5 ./CLAUDE.md

# cron-prompts.md exists?
head -5 ./cron-prompts.md
```

Tell the user:
> "Good to go. Restart the Claude session with this command:"

```bash
claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions
```
