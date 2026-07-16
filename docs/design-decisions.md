# Design Decisions & Tradeoffs

This is the "why," not the "how." [SETUP.md](../SETUP.md) tells you how to run this. This page
tells you what we chose, what we rejected, and why — the part that usually stays invisible in a
tutorial.

---

## Why Claude desktop, not a custom API integration

A custom-built integration would need its own hosting, its own auth handling per tool, and
ongoing engineering ownership for something that runs once a week. Claude's connectors already
handle OAuth to Slack/Pendo/Tability with a click, and the dashboard is an artifact Claude
publishes itself — no server to stand up or keep alive.

**Tradeoff accepted:** the routine only runs while the Claude app is open (it catches up on next
launch if it was closed). For a weekly-cadence internal tool, that's a fair trade against needing
zero infrastructure and zero engineering support for a non-technical team to run it themselves.

**Rejected:** a hosted app with its own backend and database. Overkill for the actual job — a
scheduled read-and-summarize routine — and it would have re-introduced the maintenance burden
the no-code approach is specifically avoiding.

---

## Why read-only, always

This touches a CRM with real revenue and relationship data. The fastest way to make people trust
an automated agent near that data is to guarantee it structurally *cannot* write to it — not just
promise it won't.

Two layers enforce this:
1. The MCP config exposes a single read/query tool. There's no write tool to call.
2. The optional [`advanced/`](../advanced/) guardrail blocks any write-shaped command (create /
   update / delete / upsert) at the permission layer, as a belt-and-suspenders check in case the
   underlying login ever has broader access than intended.

**Tradeoff accepted:** the agent can't log its own findings back into the CRM (e.g., can't write
a task or note). That stays a human action — deliberately, since anything that mutates a system
of record should have a person deciding to do it.

---

## Why these specific sources, and not others

| Source | What it answers |
|---|---|
| Salesforce | "When do I need to act?" — renewals, ARR, last touch |
| Pendo | "Are they actually using it?" — usage, a signal Salesforce can't give you |
| Web search | "What's happening around them?" — news, published studies, neither system has |
| Slack | Where the CS team already works — no new tool to check |
| Tability *(optional)* | Keeps an existing OKR check-in current, for teams already tracking health as a goal |

**Rejected (for now):** a support-ticket source (e.g., Zendesk). Considered for v1, deferred —
better to prove the pattern out across four sources than add a fifth before the core loop was
solid. Candidate for v2; see [iteration-notes.md](iteration-notes.md).

---

## Why signal-only, not "review every account every week"

An early version considered surfacing the full account list every week. Rejected: a fixed-size
weekly review doesn't scale as the account list grows, and a long list people skim past isn't
meaningfully different from no list at all.

Signal-only — an account only appears if it has a renewal within 90 days, a usage-risk flag, or
notable news — means a quiet week produces a short digest, and a digest that *is* long means
something is actually happening. Exception-based, not calendar-based.

**Tradeoff accepted:** this requires trusting the signal thresholds (what counts as "usage risk,"
what renewal window matters). Those got tuned after real use — see
[iteration-notes.md](iteration-notes.md).

---

## Why weekly, not daily or real-time

Renewal timing and usage trends are slow-moving. A daily run would mostly report "nothing new,"
adding noise without adding a better decision. Weekly matches how often a CS manager actually
looks at their book.

---

## Why a companion doc, not editing the account record in place

Considered writing signals directly into the existing canonical account doc. Rejected: an
automated process editing a document people treat as ground truth risks quietly introducing an
error into something they trust without checking. Chose a separate, clearly-labeled companion
doc instead — one more thing to cross-reference, but it structurally can't corrupt the source of
truth.

---

## Why one pinned dashboard link, not a fresh one every run

A new link every week means old links go stale and nobody's sure which one is current. Chose to
redeploy to one fixed address on every run. This added a step to get right during setup (see
[iteration-notes.md](iteration-notes.md) — it broke once) — but the tradeoff is a link people can
bookmark once and never have to re-share.
