# Customer Success Agent - Claude Tutorial

A simple, **no-code** way to set up a **Customer Success monitoring agent** in the
[Claude desktop app](https://claude.com/download). It watches your customer accounts across
**Salesforce, Pendo, and the web**, then publishes a **prioritized weekly dashboard** and
pings you in **Slack** - automatically.

> **Built for non-technical users.** You set everything up by clicking **Connect** inside
> Claude and signing in to your tools - no terminal, no commands. Start with [SETUP.md](SETUP.md).

---

## What it does

Once a week (we use **Thursdays at 2pm**), the agent:

1. Pulls your team's **upcoming renewals + ARR** from Salesforce (read-only).
2. Checks **Pendo** for usage risk (accounts that have gone quiet, or dormant longer than expected).
3. Searches the **web** for notable, business-relevant news and published studies per account.
4. Adds the **latest customer communication** (who / when / subject) from Salesforce.
5. Publishes a **full dashboard artifact** - a web page with all signal accounts organized by CS Manager,
   color-coded by urgency, with renewal dates, ARR, usage status, news links, and last-touch info.
   The dashboard URL stays the same every week (it updates in place).
6. Sends a **short Slack DM** - just the **top 3 most urgent accounts** and a link to the full dashboard.
7. Optionally logs a **Tability check-in** on your Customer Health OKR - accounts flagged, top
   risks, any wins - so progress tracking stays current without manual entry.

"Signal-only" means an account shows up **only if** it has: a renewal within 90 days, a
usage-risk flag, or notable news. High-value renewals always come first.

See a sample: [docs/sample-digest.md](docs/sample-digest.md).

---

## How it's built (plain version)

```
        +-------------------------------------------------+
        |             Claude (desktop app)                |
        |       runs your routine once a week             |
        +-------------------------------------------------+
          |        |        |         |          |
     read |   read |    web |   build |    send  |   log (optional)
          v        v        v         v          v          v
     Salesforce  Pendo  Web search  Artifact   Slack     Tability
      (MCP)    (connector)          dashboard (connector) (connector)
                               (stable URL)
```

Slack, Pendo, and Tability plug in through **Claude connectors** - secure links to tools your
company already uses, enabled with a click and a browser sign-in. **Salesforce is the exception:**
there's no connector for it in most orgs, so it's added with a short one-time **MCP setup** -
the only technical part (see [SETUP.md](SETUP.md), Step 3).

The dashboard artifact is built by Claude itself - no hosting, no server, no external service.
It's a web page Claude publishes to claude.ai. You link to it from Slack.

---

## Colored urgency indicators

Each account in the dashboard gets a color-coded card based on urgency:

- Red (Act Now) - renewal within 30 days, OR a high-value account with no activity in over 90 days
- Blue (Watch) - renewal 31-90 days out, OR moderate usage risk (quiet for 60-90 days)
- Green (FYI) - informational signal only (news, published study, healthy usage); no immediate risk

---

## Get started

[SETUP.md](SETUP.md) - the 15-minute, click-by-click guide (Windows and Mac).

In short:
1. Install the **Claude desktop app** and sign in.
2. **Connect** Slack and Pendo (and Tability if you use it) - one click each.
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
| [docs/sample-digest.md](docs/sample-digest.md) | Example of the weekly Slack summary and dashboard |
| [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md) | For your admin: read-only Salesforce login |
| [advanced/](advanced/) | Optional: extra client-side write-protection guardrail |

---

## Privacy

This tutorial contains **no real data** - every name, account, and ID is a placeholder or
fake example. Your real customer list and credentials stay inside your own Claude app and
connected tools; nothing sensitive lives in this repo.

---

## License

MIT - use it, adapt it, share it with your team.