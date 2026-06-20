# Advanced (optional) — extra write-protection

**You don't need this folder to run the agent.** The Salesforce MCP config in
[`salesforce/salesforce-mcp-config.json`](../salesforce/salesforce-mcp-config.json) already
limits Claude to a single **read-only** query tool, and the strongest safeguard is asking your
admin for a **read-only Salesforce login** ([../docs/salesforce-readonly-user-setup.md](../docs/salesforce-readonly-user-setup.md)).

This folder adds one optional belt-and-suspenders layer for the technically inclined:

| File | What it is |
|------|------------|
| `claude-read-only-guardrail.json` | Example permission rules that block every Salesforce **write** command (create / update / delete / upsert / import) at the tool level — a client-side safety net, in case the Salesforce login ever has more than read access. |

### How to use it
If you run Claude from a project folder, these rules can live in your project's
`.claude/settings.json`. They make Claude refuse any Salesforce write command outright,
regardless of what the connected login is allowed to do.
