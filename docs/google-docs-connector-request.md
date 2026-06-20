# (Optional) Edit-Capable Google Docs / Shared Drive Connector

An optional extension: keep an auto-updating **"Account Intel"** doc per customer in Google
Drive that refreshes weekly with news + usage + CRM data.

## The catch

A basic Google Drive connector is often **create-only** (it can make new files but not edit
existing ones) and **My Drive only** (it can't see Shared Drives). Both block "living docs":

1. **Create-only** → you can't update one doc weekly; you'd pile up duplicates.
2. **My Drive only** → it can't reach team Shared Drive folders.

## What to request from IT

A Google Workspace connector for the agent with:

- **Google Docs edit access** — ability to *update* existing documents (Docs API
  `documents.batchUpdate`), not just create.
- **Drive scope including Shared Drives** — full `drive` scope with All-Drives support, so
  docs can live in your team's Shared Drive folders.
- Authenticated as the appropriate account, or a service account granted Shared Drive access.

## Guardrails to include

- The agent should only create/edit the **auto-generated companion docs** — never the
  canonical/official account records. Scope write access to a dedicated subfolder.
- Each update is **section-replace + a "Last updated" timestamp** for an auditable trail.

## Tip: formatting

When creating a Google Doc from text, plain-text import may escape Markdown characters.
Uploading **HTML** (`content_mime_type: text/html`) converts cleanly into a formatted Doc
with real headings, bold, and links.
