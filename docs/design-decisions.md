# Design Decisions & Tradeoffs

This is the "why," not the "how." [SETUP.md](../SETUP.md) tells you how to run this. This page
tells you what we chose, what we rejected, and why - the part that usually stays invisible in a
tutorial.

---

## Why Claude desktop, not a custom API integration

A custom-built integration would need its own hosting, its own auth handling per tool, and
ongoing engineering ownership for something that runs once a week. Claude's connectors already
handle OAuth to Slack/Pendo/Tability with a click, and the dashboard is an artifact Claude
publishes itself - no server to stand up or keep alive.

**Tradeoff accepted:** the routine only runs while the Claude app is open (it catches up on next
launch if it was closed). For a weekly-cadence internal tool, that's a fair trade against needing
zero infrastructure and zero engineering support for a non-technical team to run it themselves.

**Rejected:** a hosted app with its own backend and database. Overkill for the actual job - a
scheduled read-and-summarize routine - and it would have re-introduced the maintenance burden
the zero-infrastructure approach above is specifically avoiding.

---

## Why read-only, always

This touches a CRM with real revenue and relationship data. The fastest way to make people trust
an automated agent near that data is to guarantee it structurally *cannot* write to it - not just
promise it won't.

Two layers enforce this:
1. The MCP config exposes a single read/query tool. There's no write tool to call.
2. The optional [`advanced/`](../advanced/) guardrail blocks any write-shaped command (create /
   update / delete / upsert) at the permission layer, as a belt-and-suspenders check in case the
   underlying login ever has broader access than intended.

**Tradeoff accepted:** the agent can't log its own findings back into the CRM (e.g., can't write
a task or note). That stays a human action - deliberately, since anything that mutates a system
of record should have a person deciding to do it.

**Zendesk and Clari get the read-only guarantee more directly than Salesforce did at first.**
Their custom MCP servers (see
[adding-a-custom-source.md](adding-a-custom-source.md)) simply never define a write tool for
create/update/delete/bulk-import endpoints - there's no permission layer needed to block what
was never exposed in the first place. Salesforce's belt-and-suspenders
[`advanced/`](../advanced/) guardrail exists because that connector is a general-purpose CLI
wrapper capable of writes in principle; a from-scratch server with a narrow, named tool list
doesn't have that exposure to begin with.

---

## Why these specific sources, and not others

| Source | What it answers |
|---|---|
| Salesforce | "When do I need to act?" - renewals, ARR, last touch, internal handoff notes |
| Pendo | "Are they actually using it?" - usage, a signal Salesforce can't give you |
| Zendesk | "Are they struggling?" - support-ticket volume, open count, category mix |
| Clari *(optional)* | "What does the forecast say?" - pipeline/forecast context on open opportunities |
| Web search | "What's happening around them?" - news, published studies, neither system has |
| Slack | Where the CS team already works - no new tool to check |
| Tability *(optional)* | Keeps an existing OKR check-in current, for teams already tracking health as a goal |

Zendesk and Clari were both **deferred out of v1** on purpose - better to prove the pattern out
across four sources before adding a fifth and sixth. Both shipped in v2, below, once the core
loop was solid and API access existed for each.

---

## Why add Zendesk after all

v1 deferred a support-ticket source. It got added once Zendesk API access existed, because
"are they actually using it and are they struggling with it" turned out to be two different
questions that Pendo alone couldn't answer - a dormant Pendo account and a account drowning in
support tickets look nothing alike, but both are risk.

**What made it in:** ticket count for the year, open-ticket count, and a rough category
split (billing / project-or-usage-question / software-bug / misc).

**What got dropped: CSAT.** Two reasons. First, the bulk satisfaction-ratings endpoint returned
a permissions error on the API token available - not something fixable from this side without a
higher-privilege account. Second, and more fundamental: on the actual ticket volume seen,
`priority` was essentially unset on nearly every ticket anyway, so a "highest priority" tile
would have been reporting a mostly-empty field as if it were signal. **Rule applied throughout
this project: if a field is unreliable or unavailable, drop it from the UI - don't show a
metric that looks precise but is actually mostly blank or unauthorized.**

**How matching works, and why it changed:** the obvious approach - match tickets to accounts via
the support desk's own organization field - looked right and turned out to be wrong: a live
sample found the vast majority of tickets simply weren't tagged to an organization at all,
because most support volume came through channels (web widget, chat) that don't auto-populate
it. Switched to matching by **email domain** instead - derive each account's domain(s) from its
Salesforce contacts, then match tickets by requester email domain. This is a source-agnostic
lesson worth generalizing: **don't trust a system's own "obvious" foreign key without sampling
it first** - see [iteration-notes.md](iteration-notes.md) for the numbers.

**A domain can belong to more than one account** (large organizations with several accounts on
the same email domain - think multiple divisions or agencies under one umbrella). Rather than
skip those or silently double-count, they're **flagged** with a "shared with N other accounts"
note wherever it shows up, so the number is visible with its caveat attached instead of hidden.

---

## Why Clari, and why it's optional

Clari (or an equivalent forecast/pipeline tool) answers a question none of the other sources
can: what does the forecast actually say about this account's open pipeline, independent of
what's logged in the CRM record itself. It's kept **optional and read-only, on the account
detail view only** - it does not drive urgency tiering. Reasons:

- No official MCP connector exists for it, so - like Salesforce and Zendesk - it needed the same
  custom-server treatment (see
  [adding-a-custom-source.md](adding-a-custom-source.md)). Not every team will want to do that
  setup step for a nice-to-have signal, so the whole routine is designed to work with it absent.
- Forecast data is a different kind of signal than "is this account at risk" - it's useful
  context for a conversation, not itself a renewal-risk or usage-risk flag. Folding it into the
  urgency tiers would have conflated two different judgments.
- The tool's write/ingest endpoints (bulk account or opportunity upload) were deliberately never
  wired up - same read-only principle as everywhere else in this project, applied even though
  nothing here forced the choice.

---

## Why signal-only, not "review every account every week"

An early version considered surfacing the full account list every week. Rejected: a fixed-size
weekly review doesn't scale as the account list grows, and a long list people skim past isn't
meaningfully different from no list at all.

Signal-only - an account only appears if it has a renewal within 90 days, a usage-risk flag, or
notable news - means a quiet week produces a short digest, and a digest that *is* long means
something is actually happening. Exception-based, not calendar-based.

**Tradeoff accepted:** this requires trusting the signal thresholds (what counts as "usage risk,"
what renewal window matters). Those got tuned after real use - see
[iteration-notes.md](iteration-notes.md).

---

## Why we walked back signal-only

This is the most important reversal in this project, and it happened for a concrete reason, not
a change of taste.

A usage-data refresh, run purely to fill in a few known Watch-tier cards, surfaced over a dozen
accounts with a genuine usage-risk signal that had never appeared in a single digest - because
signal-only filtering, working exactly as designed, had been silently excluding them every week.
They weren't wrong or edge cases; they simply hadn't tripped the specific thresholds the filter
checked for that week they were checked, and once excluded, an account had no path back into
view unless someone happened to re-run that specific check. **A filter that hides "nothing to
report" cannot be distinguished, by the person reading the output, from a filter that is
silently dropping things it should have caught.** Both look like a quiet week.

**What changed:** every active account gets reviewed every run, full stop. Accounts with
genuinely nothing to report land in a fourth, neutral, **collapsed-by-default** "Rest of
Accounts" group - still there, still searchable, just not competing for attention with the
tiers that need it. The three urgency tiers keep their exact original meaning; nothing about
them changed except that "no signal" now means "confirmed and shown as such," not "left out."

**Tradeoff accepted:** every run now has to actually check every account, every source, instead
of stopping early once something doesn't need investigating - more work per run, done in
exchange for the output being a true statement about the whole book instead of a curated subset
of it. For a weekly-cadence tool reviewing a few hundred accounts, that cost is worth paying;
it might not be at a much larger scale, where a genuinely stronger case for filtering would
need better observability into what's being filtered out, not just a shorter list.

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
doc instead - one more thing to cross-reference, but it structurally can't corrupt the source of
truth.

---

## Why one pinned dashboard link, not a fresh one every run

A new link every week means old links go stale and nobody's sure which one is current. Chose to
redeploy to one fixed address on every run. This added a step to get right during setup (see
[iteration-notes.md](iteration-notes.md) - it broke once) - but the tradeoff is a link people can
bookmark once and never have to re-share.

---

## Why a Heat Map tab, on top of the 360 Account View

The original single-view dashboard already showed everything a signal account needed - but only
for signal accounts, and only the fields relevant to why it was flagged. As more sources got
added, "everything about this account" stopped fitting cleanly into a signal card, and the team
using the dashboard directly (not just the person receiving the weekly digest) wanted to click
into *any* account, flagged or not, and see the full picture: Salesforce, Pendo, Zendesk,
Chatter notes, forecast context, all in one panel.

Rather than keep stretching the 360 Account View's card format to fit more sources, a second tab
was added with its own layout suited to that job - a full-book color grid plus an expandable
per-account panel. Same underlying data, different presentation for a different question ("show
me this one account, completely" vs. "show me what needs attention this week").

**Tradeoff accepted:** two tabs to maintain instead of one, and some data now gets fetched and
rendered in two different shapes. Accepted because the two tabs genuinely answer different
questions - collapsing them back into one view would mean compromising one or the other.

---

## Why a Tab-4 (CS Team Forecast) tab

Before this tab existed, the org-wide renewal and upsell forecast was assembled by hand, in
spreadsheets, pooled from what each of the roughly 15 CSMs and AMs (plus leadership reviewing the
roll-up) submitted on their own. That process had two compounding problems. First, it was
friction-heavy: reconciling 15 people's numbers into one leadership view meant a lot of back and
forth, every cycle. Second, and worse, it wasn't measuring the same thing across submitters - one
person's number was a commit figure, another's was upside, and both got pooled into the same
bucket without anyone flagging that they weren't the same kind of number. The practical effect was a
forecast that drifted from reality in both directions: what leadership expected and what
actually happened routinely showed a wide gap, positive misses and negative misses alike. The
team's own name for this was "hopecasting" - forecasting had stopped meaning "our best precise
estimate" and started meaning "what we hope happens."

Tab-4 exists to fix the two root causes directly, not just to add a chart. It defines each bucket
(Bank, Commit, Upside, Renewing, Total) with one precise, written-down rule applied identically
across all 11 CSMs, so "commit" and "upside" can no longer get silently merged the way they did
in spreadsheets. And it pulls live from Salesforce on the same weekly cadence as the rest of the
dashboard, so there's one central, real-time place everyone (CSMs, AMs, and leadership) looks at,
instead of N spreadsheets that were each a snapshot the moment they were exported.

**Tradeoff accepted:** getting the definitions right took several correction rounds against a
reference screenshot (IsWon vs. StageName, month vs. quarter scoping, Duplicate/Closed Lost
exclusions - see [iteration-notes.md](iteration-notes.md)) precisely because the bar was "matches
one exact, agreed-on definition," not "looks roughly right." That precision is the point: the
whole reason for building this was to stop the forecast from being open to per-person
interpretation.

---

## Why the report catalogs are separate artifacts, not baked into the dashboard

Two companion catalogs - a searchable index of published studies that used the product, and a
searchable index of reports done with competitor tools - get cross-referenced from the Heat
Map's per-account panel (surfacing "has this account, or one like it, published something using
us or a competitor") but are built and refreshed as their **own** artifacts, not folded directly
into the CS dashboard's data pipeline.

Reasoning: their source (a running Slack channel of shared reports, in this project's case) and
audience (also useful to sales/marketing, not just CS) are different enough that coupling their
refresh cadence and failure modes to the CS dashboard's would make both harder to reason about.
Cross-referencing them from the account panel gets the value without the coupling.
