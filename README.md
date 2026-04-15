# Claude Email Triage

An autonomous email triage agent for Claude Code that runs on a schedule, classifies unread Gmail, takes action (reply, draft, archive, spam-kill, schedule), and produces a triage report. Fully customizable — edit one skill file to match your workflow.

## What it does

- Fetches unread emails from one or more Gmail accounts
- Classifies each email (P0 urgent, P1 action, P2 review, P3 archive, spam/cold)
- Takes action based on confidence: auto-send on 95%+, draft on 70-94%, flag below 70%
- Auto-kills spam and cold outbound (replies "stop", marks spam, trashes)
- Handles meeting requests against your calendar
- Learns over time via a persistent state file (spam senders, auto-send patterns, follow-up queue)
- Produces a daily/per-run triage report

## How it runs

On a schedule you set (e.g., 8am / 12pm / 3pm / 6pm / 9pm daily), via **Claude Code Desktop's local Scheduled Tasks** feature. Runs fully unattended with the bypass-permissions setup in `SETUP.md`.

Uses the [`gog` CLI](https://github.com/steipete/gogcli) for Gmail/Calendar/Drive access. The orchestration (gog calls, scheduled tasks, state file) runs locally on your Mac, but **email content is sent to Anthropic** for Claude to read, classify, and draft replies — same as any Claude conversation. See [Privacy](#privacy) below.

## Quick start

Full setup takes ~30 minutes from scratch. See **[SETUP.md](SETUP.md)** for step-by-step instructions.

In short:
1. Install Homebrew and `gog`
2. Set up a Google Cloud project for OAuth
3. Connect `gog` to your Gmail
4. Drop the `skill/SKILL.md` into `~/.claude/scheduled-tasks/email-triage/SKILL.md`
5. Customize placeholders (accounts, VIP senders, tone, meeting preferences)
6. Enable bypass-permissions mode so it runs without prompts
7. Create the scheduled task(s) in Claude Code Desktop

## Customization

Everything lives in **one file**: `skill/SKILL.md`. It's a prompt — edit it like you'd edit instructions for a human assistant.

Placeholders to replace:
- `<YOUR_EMAIL_ALIAS>` and `<YOUR_EMAIL>` — one block per Gmail account you process
- `<VIP_SENDERS>` — people who should never be filtered
- `<YOUR_TIMEZONE>` / `<YOUR_SECONDARY_TIMEZONE>` — meeting-time display preferences
- `<YOUR_CALENDAR_LINK>` — your scheduling link (Cal.com, Google, etc.)
- `<YOUR_TONE>` — how you want drafts to sound
- `<YOUR_PROJECT_CONTEXT>` — background context for drafting replies

## Safety

The skill has hard-coded safety rules:
- Never auto-send to investors, board, or legal contacts
- Never auto-send anything about money, contracts, or commitments
- Never auto-send to people you haven't emailed before (draft instead)
- Never delete unless matching explicit delete rules; archive when unsure
- Never modify calendar events created by others
- On phishing suspicion: flag P0, do not click links

Even in bypass-permissions mode, the skill creates **drafts** for anything below 95% confidence — you review before it goes out.

## Requirements

- macOS (tested on Sonoma+)
- Claude Code Desktop app ([download](https://claude.com/claude-code))
- Claude Pro or Max plan
- Gmail account(s) with API access (free to set up)

## Privacy

Be aware of what data goes where:

- **Stays on your Mac**: the SKILL.md, the `~/email-triage/state.json` learning state, the `~/email-triage/log.md` run log, your `gog` OAuth tokens, and the cron schedule.
- **Goes to Google**: your Gmail/Calendar/Drive API requests, via `gog` (same as any Gmail client).
- **Goes to Anthropic**: the **content of every email you triage**, the prompt, the state file, and the drafts Claude generates. This is how the model classifies and writes replies — it has to read the email content. Anthropic's data handling policies for your Claude plan apply (see https://www.anthropic.com/legal).

If you have emails you don't want sent to a third-party LLM at all (NDAs, legal, medical, etc.), don't triage those accounts with this — or filter them out of the unread set before classification.

## License

MIT — see [LICENSE](LICENSE).

## Credits

Originally built for personal use across multiple Gmail accounts; generalized and open-sourced so others can run the same setup.
