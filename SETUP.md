# Setup Guide — Claude Desktop (no coding)

**You won't touch a terminal.** Everything here is done by clicking inside the **Claude
desktop app** and signing in to your tools through your browser. Plan for about **15–20
minutes**.

Works the same on **Windows and Mac**.

---

## What you need

- A work computer (Windows or Mac)
- Your normal work logins (Salesforce, Slack, Pendo, Google) — you'll just click "Connect"
  and sign in as usual
- About 15 minutes

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

## Step 3 — Add Salesforce

Salesforce is the **one** tool that often isn't preconnected. Easiest options, in order:

1. **Check your Connectors list first** (Step 2) — your org may already offer a **Salesforce**
   connector. If it's there, just **Connect** and sign in. Done.
2. **If it's not there**, ask your **Salesforce admin** to enable Salesforce's **Hosted MCP
   connector** for Claude (it's a sign-in link — no install on your side). Official guide:
   👉 https://developer.salesforce.com/docs/platform/hosted-mcp-servers/guide/claude.html
3. **Ask for read-only access.** For this routine, Claude only needs to *read* Salesforce.
   Request that the connection use a **read-only** Salesforce login so it can never change
   your CRM data. (Details your admin can follow: [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md).)

> A more technical, install-it-yourself Salesforce option exists if you ever need it — see
> [`advanced/`](advanced/). Most people won't.

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

**That's the whole setup.** Each week you'll get a prioritized summary in Slack — only the
accounts that need attention.

---

## Optional extensions (skip unless you want them)

- **Read-only Salesforce login** (recommended for peace of mind) — have your admin set it up:
  [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md)
- **Auto-updating account docs in Google Drive** — needs an edit-capable Drive connector:
  [docs/google-docs-connector-request.md](docs/google-docs-connector-request.md)
- **Technical / command-line route** (not needed for the above): [`advanced/`](advanced/)

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| A connector (Slack/Pendo/Salesforce) isn't in my list | It may not be enabled for your org yet — ask your IT/admin to turn it on. |
| The connector won't sign in | Make sure you're using your **work** account, and that pop-ups/redirects aren't blocked. |
| The routine didn't run | The Claude app needs to be open at the scheduled time; it'll run next time you open it. |
| I want to change the day/time | Just tell Claude, e.g. "change my weekly digest to Mondays at 9am." |
