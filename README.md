# Customer Success Agent — Claude Code Tutorial

A walkthrough for building a **Customer Success monitoring agent** in [Claude Code](https://claude.com/claude-code). It watches your customer accounts across **Salesforce, Pendo, and the web**, then delivers a **prioritized weekly digest** to **Slack** — automatically, on a schedule.

> **Heads up:** This is a sanitized tutorial. Every ID, email, account name, and person in here is a **placeholder or fake example**. Swap in your own values where you see `<ANGLE_BRACKETS>` or `your-...`. Nothing in this repo is a secret — see [Security & privacy](#security--privacy).

---

## What it does

Once a week (we use **Thursdays at 2pm**), the agent:

1. Pulls the team's active accounts and their **upcoming renewals + ARR** from Salesforce (read-only).
2. Checks **Pendo** for usage risk (accounts that have gone quiet).
3. Searches the **web** for notable, business-relevant news per account.
4. Pulls the **latest customer communication** (who / when / subject) from Salesforce.
5. Compiles a **signal-only, prioritized digest** — not every account, only ones that need attention — and **DMs it to you in Slack**.

"Signal-only" means an account appears **only if** it has: a renewal within 90 days, a usage-risk flag, or notable news. High-ARR renewals are always surfaced first.

---

## Architecture

```
                ┌─────────────────────────────────────────────┐
                │              Claude Code agent               │
                │        (scheduled task, runs weekly)         │
                └─────────────────────────────────────────────┘
                   │          │            │            │
          read-only│   read   │     web    │    send    │
                   ▼          ▼            ▼            ▼
            ┌───────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐
            │ Salesforce│ │ Pendo  │ │  Web     │ │  Slack   │
            │  (MCP/CLI)│ │ (MCP)  │ │  search  │ │  (MCP)   │
            │ renewals, │ │ usage  │ │  news    │ │  DM you  │
            │ ARR, comms│ │ signal │ │          │ │          │
            └───────────┘ └────────┘ └──────────┘ └──────────┘
```

Everything runs through Claude Code using **MCP connectors** (Salesforce, Pendo, Slack) plus built-in web search. No custom backend, no servers.

---

## 👋 New here? Start with the Setup Guide

**Not technical? No problem.** [**SETUP.md**](SETUP.md) is a step-by-step, copy-paste guide
for **both Windows and Mac** that installs every tool (Claude Code, Node.js, Salesforce CLI,
Git, GitHub CLI), explains what each one does, and links to official sources. Start there,
then come back here for how the agent works.

## Prerequisites (all covered in [SETUP.md](SETUP.md))

- **Claude Code** (desktop app or CLI)
- **Node.js 18+**
- **Salesforce CLI** (`@salesforce/cli`)
- **Git** + **GitHub CLI** (only if you want to share this tutorial on GitHub)
- Connectors authorized in Claude: **Salesforce**, **Pendo**, **Slack** (Google Drive optional — see [docs](docs/google-docs-connector-request.md))

---

## Setup

### 1. Install tooling

```bash
# Node.js (example via winget on Windows; use your platform's installer)
winget install OpenJS.NodeJS.LTS

# Salesforce CLI
npm install --global @salesforce/cli
```

### 2. Authenticate Salesforce (read-only intent)

```bash
sf org login web --alias your-org-alias --set-default
# verify
sf data query --query "SELECT Id, Name FROM Account LIMIT 3" --target-org your-org-alias
```

### 3. Add the Salesforce MCP server

Drop [`.mcp.json`](.mcp.json) in your project root. We lock it to a single **read-only** tool:

```json
{
  "mcpServers": {
    "salesforce": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp", "--orgs", "your-org-alias", "--tools", "run_soql_query"]
    }
  }
}
```

### 4. Apply a client-side read-only guardrail

[`.claude/settings.json`](.claude/settings.json) denies every Salesforce **write** command (create/update/delete/upsert/import, plus `apex run` / `org delete`) while allowing reads. This is a **local guardrail** — see [Read-only, done right](#read-only-done-right) for the airtight version.

### 5. (Recommended) Org-enforced read-only Salesforce user

A client-side guardrail is good; an org-enforced one is better. See [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md) for a runbook your Salesforce admin can follow (free API-only license + read-only permission set + JWT auth).

### 6. Discover your Salesforce fields

Field names differ per org. Use the Tooling API to find yours instead of guessing:

```bash
sf data query --use-tooling-api \
  --query "SELECT QualifiedApiName, Label, DataType FROM FieldDefinition WHERE EntityDefinition.QualifiedApiName='Opportunity' AND (Label LIKE '%ARR%' OR Label LIKE '%Renew%')" \
  --target-org your-org-alias
```

> **Lesson we learned:** in our org the open renewal Opportunity's ARR field was empty — the real ARR lived on the *prior* contract, reachable via a `Renewed_From__c` lookup. And prebuilt "renewal in N days" checkboxes were unreliable; filtering on the actual `Renewal_Due_Date__c` was correct. **Validate against real records before trusting a field.**

### 7. Match Pendo to Salesforce

In our setup, **Pendo accounts are keyed by the Salesforce 18-char Account ID** — so matching is exact (no fuzzy name matching). The reliable "is this account going quiet?" signal was the `metadata.auto.lastvisit` field. Confirm the equivalent in your Pendo subscription.

### 8. Create the weekly scheduled task

Use Claude Code's scheduling to run the routine every Thursday at 2pm. The full, self-contained task prompt is in [examples/weekly-digest-task.md](examples/weekly-digest-task.md) — paste it in and adjust the placeholders.

---

## Read-only, done right

This project treats "the agent must never write to our CRM" as a first-class requirement. Two layers:

| Layer | What | Strength |
|-------|------|----------|
| MCP scope | Salesforce MCP limited to `run_soql_query` | Blocks writes via the connector |
| Client guardrail | [`.claude/settings.json`](.claude/settings.json) deny rules | Blocks write commands in the shell |
| **Org-enforced** | **Dedicated read-only user** ([runbook](docs/salesforce-readonly-user-setup.md)) | **The CRM itself refuses writes — strongest** |

The first two are things you can do yourself today; the third is the audit-proof version and needs a Salesforce admin.

---

## Repo contents

| Path | What |
|------|------|
| [`SETUP.md`](SETUP.md) | **Start here** — beginner setup for Windows & Mac |
| [`.mcp.json`](.mcp.json) | Salesforce MCP config (read-only) |
| [`.claude/settings.json`](.claude/settings.json) | Client-side write guardrail |
| [`examples/accounts.sample.json`](examples/accounts.sample.json) | **Fake** account list (shape only) |
| [`examples/weekly-digest-task.md`](examples/weekly-digest-task.md) | The scheduled task prompt |
| [`docs/salesforce-readonly-user-setup.md`](docs/salesforce-readonly-user-setup.md) | Read-only SF user runbook |
| [`docs/google-docs-connector-request.md`](docs/google-docs-connector-request.md) | (Optional) Drive editing connector spec |
| [`docs/sample-slack-digest.md`](docs/sample-slack-digest.md) | Example digest output (fake data) |

---

## Security & privacy

- **No secrets in this repo.** No tokens, keys, org IDs, emails, customer names, or internal IDs. All such values are placeholders.
- **The agent is read-only** against Salesforce by design (three layers above).
- **Your real data stays local.** The actual account list, certificates, and credentials live on your machine and are **git-ignored** (see [`.gitignore`](.gitignore)) — never commit them.
- If you fork this for your team, keep your real `accounts.json`, `certs/`, and any populated configs **out of version control**.

---

## License

MIT — use it, fork it, adapt it for your team.
