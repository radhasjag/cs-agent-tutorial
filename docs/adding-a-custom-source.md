# Adding a Source With No Official Connector

Salesforce, Zendesk, and Clari all hit the same wall in this project: **no official, hosted
Claude connector exists** for any of them (at least not for this org, at the time each was
added). Rather than three different workarounds, all three used the exact same small pattern.
If you need to add a tool that isn't in the connector catalog, this is that pattern.

---

## First, check if you actually need this

Before building anything: open **Settings → Connectors** in the Claude desktop app and search
for the tool by name. Also worth a look: the tool's own product docs for "MCP" or "Model
Context Protocol" - some vendors ship an official server as an installable package (Salesforce
does, via `@salesforce/mcp` - see [SETUP.md](../SETUP.md) Step 3, which needs no custom code at
all). Only fall back to the pattern below if neither exists.

---

## The pattern

Build a small **local MCP server** - a short program that runs on your own computer, speaks the
Model Context Protocol, and translates a handful of read-only requests into calls against the
tool's real REST API. Claude talks to your server; your server talks to the vendor. Nothing
hosted, nothing to deploy - it runs only while Claude is running.

**1. Get API credentials for the tool.** Usually an API token from an account/admin settings
page, sometimes a username + token pair. Read the vendor's API docs for the auth header shape -
it varies (some want `Authorization: Bearer <token>`, some want a custom header like `apikey:
<token>`, some want HTTP Basic). Get this working with a single plain `curl`/`fetch` call before
writing any MCP code - confirms the credential and the base URL before adding a protocol layer
on top.

**2. Scaffold a server using the official SDK.** In Node:

```bash
npm install @modelcontextprotocol/sdk zod
```

**3. Expose only read tools - nothing else.** This is the important part. A minimal server looks
like this (trimmed to the shape that matters; see the SDK's own docs for the full boilerplate):

```js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const API_TOKEN = process.env.MY_TOOL_API_TOKEN;
const BASE_URL = "https://api.example-vendor.com/v1";

const server = new Server({ name: "my-tool", version: "1.0.0" });

server.tool(
  "my_tool_search_records",
  "Search records by keyword. Read-only.",
  { query: z.string() },
  async ({ query }) => {
    const res = await fetch(`${BASE_URL}/records?q=${encodeURIComponent(query)}`, {
      headers: { Authorization: `Bearer ${API_TOKEN}` },
    });
    if (!res.ok) throw new Error(`API error: ${res.status}`);
    return { content: [{ type: "text", text: JSON.stringify(await res.json()) }] };
  }
);

// Define only the read endpoints you actually need as separate tools like the one above.
// Do not define a tool for any create / update / delete / upsert / bulk-import endpoint -
// if there's no tool for it, Claude cannot call it, full stop.

new StdioServerTransport().connect(server);
```

Name each tool specifically (`my_tool_search_records`, not a generic `my_tool_request`) so it's
obvious from the tool list alone what it does and that it's read-only - a reviewer (or you,
months later) should be able to audit the whole surface just by reading the tool names.

**4. Register it in your Claude config**, the same way as the Salesforce entry in
[SETUP.md](../SETUP.md) Step 3 - `Settings → Developer → Edit Config`, add an entry pointing at
your script, with the credential in the `env` block:

```json
{
  "mcpServers": {
    "my-tool": {
      "command": "node",
      "args": ["/absolute/path/to/my-tool-mcp-server/index.js"],
      "env": { "MY_TOOL_API_TOKEN": "your-token-here" }
    }
  }
}
```

That config file lives on your own machine and is never meant to be committed anywhere - see
this repo's [`.gitignore`](../.gitignore) for the general shape of what to keep out of version
control if you're tracking your setup in git at all.

**5. Verify read-only from the outside, not just by trusting your own code.** Try asking Claude
to do something your server has no tool for (e.g. "create a new ticket") - it should say it has
no way to do that, because the tool genuinely doesn't exist. That's a stronger guarantee than a
comment saying "read-only" next to a function that could technically write.

---

## Applied three times in this project

| Tool | Why no official connector | What got exposed |
|------|---------------------------|-------------------|
| Salesforce | *(Exception - see [SETUP.md](../SETUP.md) Step 3)* an official `@salesforce/mcp` package exists, so this one skipped the custom-server step entirely - config only. | A single read/query tool. |
| Zendesk | No official MCP server published for the ticketing API at the time. | Ticket search, ticket detail, ticket comments, org/user lookup - eight to ten narrowly-scoped read tools, no write endpoints wired up at all. |
| Clari | No official MCP server published for the forecast/CRM-intelligence API at the time (a data/CRM-tooling partnership announcement existed, but no concrete endpoint to build against). | Opportunity/forecast read tools, audit-event and admin-limit read tools. The bulk data-ingest endpoints that exist on the same API were deliberately never wired up. |

If Zendesk or Clari (or any other tool) ships an official connector later, the custom server can
simply be retired in favor of it - nothing about the routine depends on the source being custom
versus official, only on the tool names it calls.
