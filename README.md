# Customer Success Agent - Claude Tutorial

A simple, **no-code** way to set up a **Customer Success monitoring agent** in the
[Claude desktop app](https://claude.com/download). It watches your customer accounts across
**Salesforce, Pendo, and the web**, then delivers a **prioritized weekly digest to Google Drive
and Slack** - automatically.

> **Built for non-technical users.** You set everything up by clicking **Connect** inside
> Claude and signing in to your tools - no terminal, no commands. Start with [SETUP.md](SETUP.md).

---

## What it does

Once a week (we use **Thursdays at 2pm**), the agent:

1. Pulls your team's **upcoming renewals + ARR** from Salesforce (read-only).
2. Checks **Pendo** for usage risk (accounts that have gone quiet, or dormant longer than expected).
3. Searches the **web** for notable, business-relevant news and published studies per account.
4. Adds the **latest customer communication** (who / when / subject) from Salesforce.
5. Writes a **full, formatted digest** to a new **Google Drive Doc** - all signal accounts with
   complete detail, organized by CS Manager.
6. Sends a **short Slack DM** - just the **top 3 most urgent accounts** and a link to the full doc.
   (Putting the whole digest in Slack got overwhelming; this keeps it actionable.)
7. Optionally logs a **Tability check-in** on your Customer Health OKR - accounts flagged, top
   risks, any wins - so progress tracking stays current without manual entry.

"Signal-only" means an account shows up **only if** it has: a renewal within 90 days, a
usage-risk flag, or notable news. High-value renewals always come first.

See a sample: [docs/sample-slack-digest.md](docs/sample-slack-digest.md).

---

## How it's built (plain version)

```
        +-------------------------------------------------+
        |             Claude (desktop app)                |
        |       runs your routine once a week             |
        +-------------------------------------------------+
          |        |        |          |          |
     read |   read |    web |     save |    send  |   log (optional)
          v        v        v          v          v          v
     Salesforce  Pendo  Web search  Google     Slack     Tability
      (MCP)    (connector)          Drive    (connector)  (connector)
                               (connector)
```

Slack, Pendo, Google Drive, and Tability plug in through **Claude connectors** - secure links
to tools your company already uses, enabled with a click and a browser sign-in. **Salesforce
is the exception:** there's no connector for it in most orgs, so it's added with a short
one-time **MCP setup** - the only technical part (see [SETUP.md](SETUP.md), Step 3).

---

## Colored urgency circles

Each account in the digest gets a colored circle based on how urgent the situation is:

- Red circle = **Act now** - renewal within 30 days, OR a high-value account with no activity in over 90 days
- Blue circle = **Watch** - renewal 31-90 days out, OR moderate usage risk (quiet for 60-90 days)
- Green circle = **FYI** - informational signal only (news, published study, healthy usage); no immediate risk

---

## Get started

[SETUP.md](SETUP.md) - the 15-minute, click-by-click guide (Windows and Mac).

In short:
1. Install the **Claude desktop app** and sign in.
2. **Connect** Slack, Pendo, and Google Drive (and Tability if you use it) - one click each.
3. **Add Salesforce** via the short MCP setup ([SETUP.md](SETUP.md), Step 3) - or hand that
   one step to a technical teammate or IT.
4. Paste the routine from [examples/weekly-digest-task.md](examples/weekly-digest-task.md) and
   ask Claude to run it weekly.

---

## Read-only by design

This agent only ever **reads** your CRM - it should never change Salesforce data. The best
way to guarantee that is to connect Salesforce with a **read-only login**. Ask your admin to
set one up using [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md).

---

## What's in this repo

| Path | What |
|------|------|
| [SETUP.md](SETUP.md) | **Start here** - step-by-step setup (mostly click-based) |
| [salesforce/salesforce-mcp-config.json](salesforce/salesforce-mcp-config.json) | Salesforce MCP config (read-only) - used in SETUP Step 3 |
| [examples/weekly-digest-task.md](examples/weekly-digest-task.md) | The routine instructions you give Claude |
| [examples/accounts.sample.json](examples/accounts.sample.json) | **Fake** account list (shows the shape) |
| [docs/sample-slack-digest.md](docs/sample-slack-digest.md) | Example of the weekly Slack summary and Google Doc |
| [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md) | For your admin: read-only Salesforce login |
| [docs/google-docs-connector-request.md](docs/google-docs-connector-request.md) | Optional: auto-updating account docs in Drive |
| [advanced/](advanced/) | Optional: extra client-side write-protection guardrail |

---

## Privacy

This tutorial contains **no real data** - every name, account, and ID is a placeholder or
fake example. Your real customer list and credentials stay inside your own Claude app and
connected tools; nothing sensitive lives in this repo.

---

## License

MIT - use it, adapt it, share it with your team.