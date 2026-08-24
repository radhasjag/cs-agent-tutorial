# Setup Guide — Claude Desktop

**Almost everything is done by clicking inside the Claude desktop app.** Salesforce needs a
short, one-time technical setup (about 10 minutes) — Zendesk and Clari need the same kind of
setup too, but they're **optional** and can be added later. If you'd rather not do the
technical part, it's the piece you can hand to a technical teammate or IT — the rest you can do
yourself. Plan for about **20–30 minutes** for the core setup (Salesforce + Pendo + Slack); add
10 minutes each, whenever you're ready, for Zendesk and Clari.

Works the same on **Windows and Mac**.

---

## What you need

- A work computer (Windows or Mac)
- Your normal work logins (Salesforce, Slack, Pendo, Google) — you'll click "Connect" and
  sign in as usual
- For the Salesforce step (and Zendesk/Clari, if you add them): the ability to install a free
  tool on your computer (or a teammate/IT who can do that step), plus an API token from each
  tool (Zendesk: Admin Center → Apps and integrations → APIs; Clari: Account Settings → API
  Token) if you're adding those

---

## Step 1 — Install the Claude desktop app

1. Go to 👉 **https://claude.com/download**
2. Download the app for your computer (Windows or Mac) and install it.
3. Open Claude and **sign in with your work account**.

> Help & docs: https://docs.claude.com/en/docs/claude-code

---

## Step 2 — Connect your tools (the easy part)

Most of your tools are **org-provided connectors** — they're already available inside Claude,
you just turn them on:

1. In Claude, open **Settings → Connectors** (sometimes shown as "Connectors" or "Add
   connectors").
2. Look for these and click **Connect** on each, then sign in through your browser when asked:
   - **Slack** — where your weekly summary gets delivered
   - **Pendo** — product-usage signals (spotting accounts going quiet)
   - **Google Drive** — *(optional)* only if you want the account-docs extension later
3. That's it for these three — no setup files, no commands.

> What's a connector? It's a secure, pre-approved link between Claude and a tool your company
> already uses. Learn more: https://docs.claude.com/en/docs/claude-code/mcp

---

## Step 3 — Add Salesforce (the one technical step)

There is **no ready-made Salesforce connector** for our organization, so Salesforce is added
through a small **MCP setup**. It's a one-time, ~10-minute job.

> 👋 **Not comfortable installing software? Hand this step to a technical teammate or IT** —
> send them this section. Everything else in this guide you can do yourself.

You'll use your computer's command window for 3b–3c: **PowerShell** on Windows (Start → type
"PowerShell") or **Terminal** on Mac (Cmd+Space → type "Terminal").

**3a. Install Node.js** — a free engine the Salesforce tool needs. Download the **LTS**
installer from 👉 **https://nodejs.org** and run it (just click through). Done once.

**3b. Install the Salesforce CLI.** In the command window, paste this and press Enter:
```
npm install --global @salesforce/cli
```

**3c. Sign in to Salesforce** (opens your browser to log in as normal):
```
sf org login web --alias your-org-alias --set-default
```

**3d. Tell Claude how to reach Salesforce.** In the Claude desktop app, open
**Settings → Developer → Edit Config** (this opens a file named `claude_desktop_config.json`).
Paste in the Salesforce server entry from
[`salesforce/salesforce-mcp-config.json`](salesforce/salesforce-mcp-config.json) — change
`your-org-alias` to the alias you used in 3c — then **save and restart Claude**.
- Reference: https://docs.claude.com/en/docs/claude-code/mcp

**3e. Read-only safety.** That config limits Claude to a single **read-only** query tool, so
it can't change Salesforce data. For an even stronger, org-enforced guarantee, ask your admin
to set up a **read-only Salesforce login**:
[docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md).

> 💡 If `npm` or `sf` says "not recognized," close the command window and open a **new** one
> after installing Node.js, then try again.

---

## Step 3b — Add Zendesk and Clari (optional, do this whenever you're ready)

Neither has a ready-made connector either, so each follows the **same pattern** as Salesforce,
just with no CLI package to install — you (or your technical teammate) write a short local
script instead. The full generic recipe, with a code template, is in
[docs/adding-a-custom-source.md](docs/adding-a-custom-source.md). In short, for each tool:

1. Get an API token (Zendesk: Admin Center → Apps and integrations → APIs → Zendesk API →
   generate a token for your own login; Clari: Account Settings → API Token — copy it
   immediately, it can't be viewed again).
2. Build the small local server following
   [docs/adding-a-custom-source.md](docs/adding-a-custom-source.md) - a handful of read-only
   tools, no write/create/delete/bulk-import endpoints wired up.
3. Add the entry to the same `claude_desktop_config.json` from Step 3d, using
   [`zendesk/zendesk-mcp-config.json`](zendesk/zendesk-mcp-config.json) or
   [`clari/clari-mcp-config.json`](clari/clari-mcp-config.json) as the template - fill in your
   own subdomain/token, point `args` at wherever you saved the script, then restart Claude.

**Skip this step entirely if you don't have API access to Zendesk or Clari yet** - the weekly
routine works fine with just Salesforce + Pendo, and you can add either tool later without
redoing anything else.

---

## Step 4 — Create your weekly routine

No code — you just tell Claude what you want and it sets up a recurring task.

1. Open [`examples/weekly-digest-task.md`](examples/weekly-digest-task.md) and copy the
   instructions inside (fill in the few `<PLACEHOLDERS>` — like your CS managers' names and
   your Slack DM).
2. In Claude, paste them and say something like:

   > "Run these instructions every **Thursday at 2pm** and send the result to my Slack DM."

3. Claude saves it as a **scheduled routine** that runs automatically.

> ℹ️ The routine runs while the Claude app is open. If the app is closed when it's due, it
> runs the next time you open it.

**That's the whole setup.** Each week you'll get a Slack ping and a dashboard covering your
whole book — see [README.md](README.md#three-views-one-dashboard) for what the dashboard's
three tabs each show.

---

## Optional extensions (skip unless you want them)

- **Zendesk and Clari** — see Step 3b above; add whenever you have API access to either.
- **Read-only Salesforce login** (recommended for peace of mind) — have your admin set it up:
  [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md)
- **Auto-updating account docs in Google Drive** — needs an edit-capable Drive connector:
  [docs/google-docs-connector-request.md](docs/google-docs-connector-request.md)
- **Extra Salesforce write-protection** (a client-side guardrail, on top of read-only): [`advanced/`](advanced/)
- **Report catalogs** (published studies / competitor-tool reports, cross-referenced from the
  Heat Map tab) — optional, built the same no-code way; ask Claude to set one up from a Slack
  channel or spreadsheet of reports if useful to your team.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| A connector (Slack/Pendo/Salesforce) isn't in my list | It may not be enabled for your org yet — ask your IT/admin to turn it on. |
| The connector won't sign in | Make sure you're using your **work** account, and that pop-ups/redirects aren't blocked. |
| The routine didn't run | The Claude app needs to be open at the scheduled time; it'll run next time you open it. |
| I want to change the day/time | Just tell Claude, e.g. "change my weekly digest to Mondays at 9am." |
| A Zendesk/Clari tool call fails with an auth error | Double-check the token in `claude_desktop_config.json`'s `env` block, and that you restarted Claude after editing it. |
| I don't have Zendesk/Clari access yet | Skip Step 3b entirely — everything else works without them. |
