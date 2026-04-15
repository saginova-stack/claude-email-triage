# Email Triage — Personal Config

This is where your personal settings live. The `SKILL.md` reads this file at the start of every run. Because your settings are here and not in `SKILL.md`, you can pull upstream improvements to the skill without losing your customization.

**Copy this file to `~/email-triage/config.md` and edit the values below.**

```bash
cp config.example.md ~/email-triage/config.md
```

---

## Accounts

List every Gmail account you want the agent to triage. The `alias` must match an alias you set up with `gog auth alias set <alias> <email>`.

| Alias | Email                        | Context                                           |
|-------|------------------------------|---------------------------------------------------|
| work  | you@yourcompany.com          | Work account: clients, partners, team             |
| pers  | you@gmail.com                | Personal: network, intros, friends                |

Remove rows you don't need. Add more as you have more accounts.

---

## VIP Safe Senders

These senders should ALWAYS pass through as P0 or P1. Never mark spam, never auto-archive, never auto-handle. Add entries as you identify VIPs.

- your-boss@yourcompany.com
- your-cofounder@yourcompany.com
- your-spouse@example.com

---

## Meeting Preferences

How the agent should handle meeting-request emails.

- **Default meeting length**: 30 minutes
- **Primary timezone** (yours): `America/Los_Angeles` (e.g., `Europe/Amsterdam`, `America/New_York`)
- **Display timezones** (show these to senders): `PST, EST`
- **Earliest time you'll take meetings**: 9:00 AM local
- **Latest time you'll take meetings**: 6:00 PM local
- **Buffer between meetings**: 15 minutes
- **Default calendar link**: https://cal.com/yourname/30min

### Meeting-type shortcuts (optional)

For different call types with different lengths or links:

- **Sales / prospect calls**: 45 min, https://cal.com/yourname/45min
- **Intro / networking calls**: 30 min, https://cal.com/yourname/30min
- **Client calls**: 30 min, https://calendar.app.google/yourclientlink

---

## Tone and Style

How drafts should sound in your voice. Claude will mimic these preferences when generating replies.

- Professional but warm, not corporate
- Brief and direct — short emails, not long
- Sign off with just first name (no title, no company, no phone)
- Never start with "Hope this email finds you well" or similar filler
- When uncertain about tone, err on the side of shorter
- No em dashes in outbound email. Use commas, periods, parentheses.
- English only. Do not draft in other languages.

(Replace these lines with your own preferences. Leave or remove each bullet as you like.)

---

## Project Context

Background the agent uses when drafting replies. The more specific, the better drafts will be.

**Who you are:** Your name, your role (e.g., "Jane Doe, founder of Acme Co.")

**What you work on:** One or two lines describing your company/work so Claude can reference it intelligently in replies to prospects and partners.

**Primary contacts:**
- **Clients / customers**: description of who they are and how you talk to them
- **Investors / board**: hands-off, formal; always P0 priority; never auto-send
- **Team**: casual
- **Network / community**: warm intros, always P1

**Sensitive topics — always flag, never auto-send:**
- Contracts, pricing negotiations, legal
- Equity / cap table / fundraise
- Anything involving money movement
- Medical / personal / confidential

**Known automated notification senders to auto-handle:**
(These are services that send you routine notifications you can delete or archive without reading.)
- Stripe receipts: delete
- GitHub notifications for repos you own: archive to a label, don't inbox
- Calendar invite responses: archive after reading
- Google Workspace admin alerts: review, then archive

---

## Run Frequency (for reference — set the actual schedule in Claude Code Desktop UI)

Typical schedule:
- Morning: 8:00 AM
- Midday: 12:00 PM
- Afternoon: 3:00 PM
- Early evening: 6:00 PM
- Night: 9:00 PM

Each run processes unread emails since the previous run (or last 8 hours, whichever is shorter).
