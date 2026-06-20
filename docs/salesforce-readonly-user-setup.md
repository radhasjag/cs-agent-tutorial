# Read-Only Salesforce Integration User — Runbook

**Goal:** Give the agent a Salesforce login that the org *physically prevents* from writing
any data — the audit-proof version of read-only, enforced by Salesforce rather than by
client config.

**Design:** API-only integration license + API-only profile + a custom **read-only**
permission set + **JWT certificate** authentication (no password anywhere).

> Replace `<YOUR_ORG_ALIAS>`, `<INTEGRATION_USERNAME>`, and `<CONSUMER_KEY>` with your values.
> Never commit the generated private key.

---

## Part 1 — Salesforce Admin tasks (Setup UI)

These require admin rights and cannot be scripted from the CLI.

1. **Read-only permission set** (e.g. `CS Agent Read Only`, License: None). Grant **Read**
   only (no Create/Edit/Delete) on the objects the agent reads — typically Account, Contact,
   Contract, Opportunity, Order, Case, User. Enable **API Enabled** (and **Run Reports** if
   the agent reads a report). Do NOT grant any write/Modify All.
2. **Integration user** on a free **API-only integration license**, with an **API-only profile**
   (commonly "Minimum Access - API Only Integrations").
3. **Assign** the read-only permission set to that user.
4. **Connected App** with OAuth + **"Use digital signatures"** — upload the public certificate
   (`server.crt`, generated below). Scopes: *Manage user data via APIs (api)* and
   *Perform requests at any time (refresh_token, offline_access)*.
5. **Pre-authorize** the user: in the Connected App policies set "Admin approved users are
   pre-authorized", relax IP restrictions, and authorize the permission set/profile.

## Part 2 — Generate the JWT certificate (you)

```bash
openssl genrsa -out server.key 2048
openssl req -new -x509 -nodes -sha256 -days 730 -key server.key \
  -out server.crt -subj "/CN=cs-agent-readonly/O=YourCompany"
```

- Hand the admin **`server.crt`** (public — safe to share).
- Keep **`server.key`** private. **Never commit it** (it's in `.gitignore`).

## Part 3 — Authenticate the CLI (you, after Part 1)

```bash
sf org login jwt \
  --username <INTEGRATION_USERNAME> \
  --jwt-key-file server.key \
  --client-id <CONSUMER_KEY> \
  --alias cs-agent-ro --set-default
```

Then point `.mcp.json` `--orgs` at `cs-agent-ro`.

## Part 4 — Verify it's truly read-only

- Run a `sf data query` → should succeed.
- Attempt any write → the org should reject it with an INSUFFICIENT_ACCESS error.
  That rejection is your proof for a compliance review.

## Notes
- This user is API-only: it cannot log into the Salesforce web UI, and with the permission
  set above it cannot write — via the CLI, MCP, or any other tool.
- Rotate the certificate before it expires.
