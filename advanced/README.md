# Advanced (optional) — technical / command-line route

**Most people don't need anything in this folder.** The main [SETUP.md](../SETUP.md) gets you
running entirely through the Claude desktop app and connectors.

These files are here only for technically comfortable users or admins who prefer to run
Salesforce through a locally-installed MCP server instead of a connector.

| File | What it is |
|------|------------|
| `salesforce-mcp-config.json` | Example config for running the `@salesforce/mcp` server locally (requires Node.js + the Salesforce CLI). Restricts the agent to a single **read-only** query tool. |
| `claude-read-only-guardrail.json` | Example permission rules that block all Salesforce **write** commands at the tool level — a client-side safety net for the command-line route. |

### When you'd use this
- Your org can't enable a Salesforce connector and you'd rather self-host the MCP server.
- You want an extra client-side guardrail on top of a read-only Salesforce login.

### The far simpler alternative
Use a Salesforce **connector** in Claude (see [SETUP.md](../SETUP.md), Step 3) and ask your
admin for a **read-only** Salesforce login ([../docs/salesforce-readonly-user-setup.md](../docs/salesforce-readonly-user-setup.md)).
That achieves the same read-only safety with no installation.
