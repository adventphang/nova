# Nova — AI Assistant on Claude Code

Nova is a 24/7 personal AI assistant that runs on your server and communicates with you over Telegram. It monitors itself, runs scheduled tasks (daily briefing, health checks, cron watchdog), and exposes a web dashboard at `http://localhost:7777/ui`.

By the end of this guide you will have Nova running in a persistent tmux session on an Ubuntu 24.04 VPS, sending you a welcome message on Telegram.

---

## Phase 1 — Prerequisites

Install the system packages Nova depends on:

```bash
sudo apt update && sudo apt install -y \
  python3 python3-venv python3-pip \
  git curl jq tmux \
  nodejs npm
```

Verify everything is in place before continuing:

```bash
node --version       # v18 or higher
python3 --version    # 3.10 or higher
git --version
which curl jq tmux
```

All commands should print a version or path. If any fail, re-run the install step above.

---

## Phase 2 — Telegram Bot

Nova communicates with you through a Telegram bot that you own. Create one now:

1. Open Telegram and search for **@BotFather**.
2. Start a chat and send `/newbot`.
3. When prompted for a **name**, enter something like `Nova` — this is the display name users see.
4. When prompted for a **username**, enter something unique ending in `bot`, e.g. `my_nova_bot`.
5. BotFather replies with a message containing your **API token** — a string that looks like `123456789:ABCDefgh...`.

Copy that token and keep it somewhere safe. You will need it in Phase 3.

---

## Phase 3 — Claude Code

### 3.1 Install

Follow the [official Claude Code installation guide](https://docs.anthropic.com/en/docs/claude-code/getting-started). Return here once this works:

```bash
claude --version
```

### 3.2 Configure

These settings are required for Nova. Apply them once — they persist globally.

Start a `claude` session, then apply the following two settings inside it:

**Configure the status line** to show current folder, model, and colour-coded context usage. Run `/status-line` and paste this prompt when asked what to display:

```text
Show the current folder name, the active model, and context usage as a colour-coded progress bar with a percentage value.
```

**Install plugins** using slash commands:

```text
/plugins install context7@claude-plugins-official
/plugins install telegram@claude-plugins-official
```

When the Telegram plugin prompts for a bot token, paste the token from Phase 2.

You can now exit the `claude` session. Apply the remaining settings from your terminal:

**Disable auto-memory** (Claude Code would otherwise generate memory files you don't want):

```bash
claude config set -g autoMemoryEnabled false
```

**Enable agent teams** (required for Nova's multi-agent cron jobs). Add this to your `~/.bashrc` so it persists across logins:

```bash
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.bashrc
source ~/.bashrc
```

---

## Phase 4 — Install Nova

Create a directory for the assistant and clone this repository into it:

```bash
mkdir ~/nova && cd ~/nova
git clone <this-repo-url> .
```

Start Claude Code:

```bash
claude
```

Once the session is open, paste this prompt exactly:

```text
Read the file SETUP.md and follow every step in it to set up a 24/7 AI assistant. Ask me for confirmation before each major step.
```

Claude Code will walk you through the setup interactively. It will ask for:

- **Assistant name** — what Nova should call itself (e.g. `Nova`)
- **Personality** — describe the tone you want (e.g. `dry wit, concise, no fluff`)
- **Timezone** — your city or IANA timezone (e.g. `Kuala Lumpur`)

Answer each prompt and confirm each major step when asked.

### Verify the installation

Once setup completes, run these checks:

```bash
# Backend API running?
curl -s http://127.0.0.1:7777/health
# Expected: {"status":"ok"}

# System prompt written?
head -5 ./CLAUDE.md

# Cron jobs persisted?
head -5 ~/nova/cron-prompts.md
```

All three should return output. If the health check fails, the setup step for the backend API may not have completed — re-run it from the Claude Code session.

---

## Phase 5 — Run

Start a named tmux session so Nova keeps running after you disconnect:

```bash
tmux new -s nova
```

Inside the tmux session, launch Nova:

```bash
claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions
```

Nova will perform its startup checks, recreate its cron jobs, and send you a welcome message on Telegram.

**tmux essentials:**

| Action                 | Key / Command         |
|------------------------|-----------------------|
| Detach (leave running) | `Ctrl+B` then `D`     |
| Reattach later         | `tmux attach -t nova` |
| List sessions          | `tmux ls`             |

---

## Phase 6 — Maintenance

Clear Nova's context window regularly to keep responses sharp and avoid hitting limits. The recommended triggers are:

- **Nightly** — clear before going to bed each night.
- **At 40% usage** — clear whenever the status line shows the context bar approaching 40%.

To clear, reattach to the tmux session and run one of these inside the Claude Code session:

| Command    | Effect                                         |
|------------|------------------------------------------------|
| `/clear`   | Wipes the context window entirely              |
| `/compact` | Summarises history into a compressed context   |

---

## Phase 7 — Expanding capabilities

The default setup comes with four cron jobs:

1. **Cron watchdog** — Claude session cron jobs expire after 7 days. This job automatically renews any that are about to lapse.
2. **Backend API health** — Periodically checks whether the backend service is running and restarts it if it is down.
3. **Heartbeat** — Verifies that all scheduled jobs are active, recreates any that are missing, and sends you an occasional status update.
4. **Daily briefing** — Sends you a morning message covering the weather, exchange rates, news, and upcoming films — delivered at 9 AM.

From here on, the world is your oyster. Just tell your bot on Telegram what capability you would like to add.

### Example 1: Integrate with a Microsoft work account

```text
Integrate with my Microsoft 365 work account using Graph API. The tenant ID is `xxx` and the client ID is `xxx`. We will use device flow login. You must save the refresh token to exchange for a fresh access token when it expires. You will use this to read my calendars and emails.
```

### Example 2: Customise the daily briefing

```text
Update my daily briefing. I want to know all my commitments for the day — pull them from my Microsoft work calendar. Also include USDMYR, SGDMYR, MYRIDR, BTCUSD, and S&P 500 index in your briefing. I also want bizarre or funny news stories (nothing serious, please). Deliver my briefing at 8:17 AM sharp each day.
```
