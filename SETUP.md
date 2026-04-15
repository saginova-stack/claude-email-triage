# Setup Guide — Claude Email Triage

Start to finish on a fresh Mac. Total time: ~30 minutes, most of which is Google Cloud OAuth setup (one-time).

After setup, the agent runs on a schedule (e.g., 5× daily) **fully unattended** — no permission prompts, no babysitting.

---

## What you're building

1. **`gog`** — a local CLI that gives Claude Code read/write access to your Gmail, Calendar, and Drive
2. **A SKILL.md** — the prompt that tells Claude how to triage your email
3. **Claude Code's Scheduled Tasks** — runs the skill on a cron schedule
4. **Bypass-permissions mode** — lets Claude run tools without asking each time

---

## Prerequisites

- A Mac (tested on macOS Sonoma and later)
- **Claude Code Desktop app** — download from https://claude.com/claude-code
- **Claude Pro or Max plan** (scheduled tasks require a paid plan)
- One or more Google accounts you want to triage

---

## Part 1: Install Homebrew (skip if you already have it)

Open Terminal (search "Terminal" in Spotlight) and run:

```bash
brew --version
```

If it says "command not found", install Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the on-screen instructions. When it finishes, close and reopen Terminal.

---

## Part 2: Install `gog` (1 min)

`gog` is the command-line tool that gives Claude access to your Google Workspace.

```bash
brew install steipete/tap/gogcli
```

Verify it works:

```bash
gog --version
```

---

## Part 3: Google Cloud OAuth setup (10 min, one-time)

You need to create your own Google Cloud project so `gog` can access Gmail on your behalf. This is free and takes about 10 minutes.

### 3.1 Create a Google Cloud project

1. Go to https://console.cloud.google.com/projectcreate
2. Sign in with the Google account you want to triage
3. Project name: `Email Triage` (or anything)
4. Click **Create**

### 3.2 Enable the Gmail, Calendar, and Drive APIs

Enable each of these (select your project in the top dropdown first, then click **Enable**):

- Gmail API — https://console.cloud.google.com/apis/api/gmail.googleapis.com
- Calendar API — https://console.cloud.google.com/apis/api/calendar-json.googleapis.com
- Drive API — https://console.cloud.google.com/apis/api/drive.googleapis.com

### 3.3 Configure OAuth consent screen

1. Go to https://console.cloud.google.com/auth/branding
2. User type: **External**
3. App name: `Email Triage`
4. User support email: your email
5. Developer contact email: your email
6. Click **Save**

### 3.4 Add yourself as a test user

1. Go to https://console.cloud.google.com/auth/audience
2. Click **Add users**
3. Enter the Google account(s) you want to triage
4. Click **Save**

### 3.5 Create OAuth credentials

1. Go to https://console.cloud.google.com/auth/clients
2. Click **Create Client**
3. Application type: **Desktop app**
4. Name: `gog`
5. Click **Create**
6. Click **Download JSON**
7. Save the file to your Downloads folder

---

## Part 4: Connect `gog` to your account(s) (3 min)

### 4.1 Store the credentials

```bash
gog auth credentials ~/Downloads/client_secret_*.json
```

### 4.2 Add each Google account

For each account you want to triage, run:

```bash
gog auth add your-email@example.com --services gmail,calendar,drive
```

A browser window opens. Sign in, accept the "Google hasn't verified this app" warning (it's your own app), and grant all requested scopes.

Repeat for each additional account.

### 4.3 Set short aliases

Aliases make commands readable. Pick a 2-3 letter alias per account:

```bash
gog auth alias set work your-work-email@example.com
gog auth alias set pers your-personal-email@gmail.com
```

### 4.4 Test it

```bash
gog --account work gmail search "is:unread" --max 3
```

If you see emails listed, it's working.

---

## Part 5: Set up the skill files (3 min)

### 5.1 Create the state/log directory

**Important:** this directory must be **outside** `~/.claude/`. Claude's bypass-permissions mode still prompts for writes inside `~/.claude/`, so keeping state elsewhere is required for fully unattended runs.

```bash
mkdir -p ~/email-triage
```

### 5.2 Create the initial state file

```bash
cat > ~/email-triage/state.json << 'EOF'
{
  "last_run": null,
  "spam_senders": [],
  "cold_outbound_patterns": [],
  "draft_edits": [],
  "auto_send_log": [],
  "follow_up_queue": [],
  "unsubscribed": [],
  "stats": {
    "total_processed": 0,
    "auto_sent": 0,
    "drafts_created": 0,
    "archived": 0,
    "spam_killed": 0,
    "follow_ups_sent": 0
  }
}
EOF
```

### 5.3 Drop the skill into place

```bash
mkdir -p ~/.claude/scheduled-tasks/email-triage
cp skill/SKILL.md ~/.claude/scheduled-tasks/email-triage/SKILL.md
```

(Adjust the `cp` source path if you cloned this repo somewhere other than your current directory.)

### 5.4 Customize the skill

Open `~/.claude/scheduled-tasks/email-triage/SKILL.md` in your editor and search for `<` to find every placeholder. At minimum:

- **`Accounts` section**: one row per Gmail alias you configured in Part 4.3
- **`VIP Safe Senders`**: email addresses that should never be filtered
- **`Tone and Style`**: how drafts should sound in your voice
- **`Meeting preferences`**: default length, your calendar link, your timezone
- **`Project Context`**: who you are, what you work on (so Claude can draft relevantly)

Skip anything that doesn't apply to you (e.g., if you don't do meetings, leave that section alone — the skill handles absent preferences gracefully).

---

## Part 6: Enable bypass-permissions mode (2 min)

This is the step that makes the skill run **without asking you to approve every Gmail command**.

Open `~/.claude/settings.json` (create it if it doesn't exist):

```bash
open -e ~/.claude/settings.json
```

Add these two keys. Merge with anything already there:

```json
{
  "skipDangerousModePermissionPrompt": true,
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

**What this does:**
- `defaultMode: "bypassPermissions"` — Claude Code sessions start in bypass mode by default, so scheduled tasks don't prompt for tool calls
- `skipDangerousModePermissionPrompt: true` — skips the "are you sure you want bypass mode?" dialog at session start

**Safety trade-off:** bypass mode is unrestricted. Only use it on your personal machine with skills you trust. The skill itself has built-in safety rules (never auto-send to investors, always draft for <95% confidence, etc.) — see the Safety Rules section of `SKILL.md`.

Enable Claude Code Desktop's keep-awake setting (Preferences → enable "Keep Awake") so scheduled tasks fire even when your screen is asleep.

---

## Part 7: Test manually (5 min)

Open Claude Code Desktop and start a new session. Paste:

> Run the email triage skill at `~/.claude/scheduled-tasks/email-triage/SKILL.md`. Process all accounts. Show me the full triage report. Do NOT auto-send anything on this first run — create drafts only so I can review.

Review the output carefully:
- Are spam/cold emails correctly identified?
- Are drafts reasonable in tone?
- Are VIP senders flagged P0/P1?
- Is the state file at `~/email-triage/state.json` getting updated?

If it looks right, loosen the "no auto-send" constraint and test again.

---

## Part 8: Create the scheduled task (3 min)

Open the **Claude Code Desktop** app:

1. Click **Schedule** in the sidebar
2. Click **+ New task**
3. Choose **Local task**
4. Fill in:
   - **Name**: `email-triage-morning` (or whatever)
   - **Description**: `Triage my inbox`
   - **Prompt source**: point at the SKILL.md you just installed, OR paste the contents directly
   - **Model**: Sonnet 4.6 is a good default (Opus if you want higher-quality drafts)
   - **Frequency**: Daily at your chosen time (e.g., 8:00 AM)
5. Click **Create task**

---

## Part 9: Multiple run-times per day (optional)

Claude Code's scheduled-tasks feature is **one folder = one cron expression**. To run 5× daily you need 5 separate task folders.

Rather than duplicating the SKILL.md five times, create one canonical file and symlink the rest:

```bash
cd ~/.claude/scheduled-tasks
mkdir -p email-triage-12pm email-triage-3pm email-triage-6pm email-triage-9pm

# Canonical file lives in email-triage/ (from Part 5.3)
# Others are symlinks
ln -s ../email-triage/SKILL.md email-triage-12pm/SKILL.md
ln -s ../email-triage/SKILL.md email-triage-3pm/SKILL.md
ln -s ../email-triage/SKILL.md email-triage-6pm/SKILL.md
ln -s ../email-triage/SKILL.md email-triage-9pm/SKILL.md
```

Then create a scheduled task in the UI for each folder (Part 8), each with its own frequency. Edit the canonical file once; all of them pick it up.

---

## Part 10: Verify it's running unattended

After the first scheduled run:

```bash
# Check that state is being updated
cat ~/email-triage/state.json | grep last_run

# Skim the run log
tail -100 ~/email-triage/log.md
```

If either is empty, see Troubleshooting.

---

## Troubleshooting

### Scheduled task fires but asks me to approve tool calls

The most common cause: `defaultMode: "bypassPermissions"` isn't set (or is set in the wrong place).

Check `~/.claude/settings.json`:
- It must be `permissions.defaultMode`, nested — **not** top-level `defaultMode`
- The JSON must be valid (a single trailing comma breaks the whole file)

Verify:

```bash
cat ~/.claude/settings.json | python3 -m json.tool
```

If Python prints the JSON back cleanly, the file is valid.

### State/log writes are still prompting

Make sure `state.json` and `log.md` are at `~/email-triage/`, **not** `~/.claude/email-triage-*`. Bypass mode deliberately still prompts for writes inside `~/.claude/` for safety.

### Per-task `allowedTools` in `~/.claude/projects/...` doesn't work

The top-level `allowedTools` key is legacy/unsupported. If you need per-task permissions, use nested `permissions.allow` instead:

```json
{
  "permissions": {
    "allow": ["Bash(gog:*)"]
  }
}
```

That said: once bypass mode is on globally, you usually don't need per-task permission rules at all.

### `gog: command not found`

Close Terminal and reopen it. If still broken:

```bash
export PATH="/opt/homebrew/bin:$PATH"
gog --version
```

Add the `export` line to your `~/.zshrc` permanently.

### OAuth says "Access blocked"

Make sure you added your email as a test user in Part 3.4, and that you enabled all three APIs (Gmail, Calendar, Drive) in Part 3.2.

### Scheduled task doesn't fire at all

- Mac must be awake at the scheduled time. Enable "Keep Awake" in Claude Code Desktop preferences, or use an app like Amphetamine.
- Claude Code Desktop must be **running** (can be in the background, just not quit).
- Check the task's status in the Schedule sidebar — it will show the last run time.

### Agent misclassifies emails

The agent learns over time via `~/email-triage/state.json`. You can also directly edit the state file:
- Add senders to `spam_senders` to auto-kill future messages
- Add senders to `cold_outbound_patterns` to auto-draft (not auto-send) polite declines
- Edit the VIP Safe Senders list inside `SKILL.md` for senders that should never be filtered

---

## How to update the skill

When you want to refine the agent's behavior:

1. Edit `~/.claude/scheduled-tasks/email-triage/SKILL.md`
2. Save
3. Next scheduled run picks up the change automatically — no restart needed

If you set up multi-schedule with symlinks (Part 9), you only need to edit the canonical file.

---

## Security and privacy notes

**What stays on your Mac:**
- The SKILL.md prompt
- `~/email-triage/state.json` (learning state) and `log.md` (run log)
- `gog` OAuth tokens (stored in your macOS keychain)
- The cron schedule (Claude Code task metadata)

**What goes to Google:**
- Gmail/Calendar/Drive API requests via `gog` — same as any Gmail client.

**What goes to Anthropic:**
- The **full content of every email you triage**, the SKILL.md prompt, your state file contents, and the drafts Claude generates. The model has to read the email content to classify and reply. Anthropic's data handling for your Claude plan applies — see https://www.anthropic.com/legal.

**Practical implications:**
- Don't run this against accounts containing data you don't want sent to a third-party LLM (NDAs, legal, medical, privileged client communication).
- The state file contains a log of senders and what was auto-sent/drafted. Back it up if you want, but treat it as private.
- Bypass-permissions mode is **per-machine**, not synced. Enabling it on one Mac doesn't enable it anywhere else.
- This repository contains **no credentials** and **no personal data**. The SKILL.md ships with placeholders only.

---

## Feedback / issues

Open an issue at https://github.com/saginova-stack/claude-email-triage
