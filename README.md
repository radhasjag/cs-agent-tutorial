# Customer Success Agent - a Claude Playbook

A **no-code** way to set up a **Customer Success monitoring agent** in the
[Claude desktop app](https://claude.com/download) - and the documented playbook behind it: not
just the setup steps, but why each choice was made, what broke running it for real, and what's
next. It watches your customer accounts across **Salesforce, Zendesk, Pendo, Clari, and the
web**, then publishes a **multi-view dashboard** and pings you in **Slack** - automatically.

> **Want to just run it?** Everything below still works as a **click-through, no-code setup** -
> no terminal, no commands. Start with [SETUP.md](SETUP.md).
>
> **Want the reasoning?** See [Results & Impact](#results--impact) below, then
> [docs/design-decisions.md](docs/design-decisions.md) and
> [docs/iteration-notes.md](docs/iteration-notes.md).

> **v2 update:** this started as a 4-source, signal-only weekly digest (see the original
> [v1 write-up](#v1-recap) below). Running it for real - for months, against a full account
> book - pushed it into something bigger: two more data sources, a full per-account view that
> pulls from every corner of every connected system, and a walk-back of the original
> "only show me what needs attention" design once that filtering turned out to be hiding real
> risk. That evolution is the real story here - see [What changed in v2](#what-changed-in-v2).

> **v3 update:** a fourth tab, **CS Team Forecast**, added an org-wide view for a different
> audience: not "what does one CSM need to act on," but "what does the whole team's renewal and
> upsell forecast add up to, right now, on one shared definition." See
> [What changed in v3](#what-changed-in-v3).

---

## Why this exists

Two versions of the same underlying problem, solved the same way.

**Weekly account visibility.** A manual pass across a CRM, a support desk, a usage-analytics
tool, and outside news, repeated for every account, every week, does not survive contact with a
busy quarter - not from lack of effort, but because cross-referencing four systems by hand gets
harder every time you add a fifth, and a missed account looks identical to a quiet one until
something breaks (see [docs/iteration-notes.md](docs/iteration-notes.md)).

**Org-wide forecasting.** Before the CS Team Forecast tab existed, the whole team's renewal and
upsell number was pooled by hand, in spreadsheets, from roughly 15 CSMs and AMs - then rolled up
for leadership. That process didn't just cost time; it wasn't measuring the same thing across
submitters. One person's number was a commit figure, another's was upside, and both landed in the
same bucket with no flag that they weren't equivalent. The team's own name for the result was
**"hopecasting"** - a forecast that routinely missed reality in both directions, because it was
never built on one consistent definition to begin with.

Both problems have the same fix: **one automatically-refreshed, precisely-defined source of
truth**, instead of a manual reconstruction that degrades under load or drifts across whoever's
compiling it that week. Four dashboard tabs exist because four different people ask four
different questions of that same underlying data - see
[Four views, one dashboard](#four-views-one-dashboard) below.

This wasn't assembled by pointing an AI at some connected tools and taking whatever came out.
Every choice below - what stays read-only and why, which four tabs and why not one or five, why a
filtering approach that looked reasonable at first got reversed months in, why the forecast
tab's field-level definitions went through several correction rounds against a reference before
they were trusted - was made deliberately, run against a live account book for months, and
corrected when real usage proved an assumption wrong.
[docs/design-decisions.md](docs/design-decisions.md) and
[docs/iteration-notes.md](docs/iteration-notes.md) are the record of that thinking, not
documentation written after the fact to justify it.

---

## What it does

Once a week (we use **Thursdays at 2pm**), the agent:

1. Pulls your team's **upcoming renewals + ARR** from Salesforce (read-only).
2. Checks **Pendo** for usage risk (accounts that have gone quiet, or dormant longer than
   expected) - including accounts that only look dormant because their real usage happens
   through a reseller, partner, or a child/sub-account.
3. Pulls **support ticket volume, open-ticket count, and category mix** from Zendesk, matched
   to each account even when the support desk itself doesn't tag tickets by organization.
4. Optionally checks **Clari** for forecast/pipeline signal on the account's open opportunities.
5. Searches the **web** for notable, business-relevant news and published studies per account.
6. Pulls the **latest customer communication** (who / when / subject) from Salesforce, plus any
   recent internal **handoff note** left in Salesforce Chatter (e.g. "watch this while I'm out").
7. Publishes a **multi-view dashboard artifact** - see [Four views, one dashboard](#four-views-one-dashboard)
   below. The dashboard URL stays the same every week (it updates in place).
8. Sends a **short Slack DM** - just the **top 3 most urgent accounts** and a link to the full
   dashboard.
9. Optionally logs a **Tability check-in** on your Customer Health OKR - accounts flagged, top
   risks, any wins - so progress tracking stays current without manual entry.

Every account in the book gets reviewed, every run - see
[Why we walked back "signal-only"](docs/design-decisions.md#why-we-walked-back-signal-only) for
why that changed from the original design.

See a sample: [docs/sample-slack-digest.md](docs/sample-slack-digest.md).

---

## Four views, one dashboard

The dashboard is one Claude artifact with four tabs, each answering a different question, for a
different audience:

| Tab | Answers | Built from | Mainly for |
|-----|---------|------------|------------|
| **360 Account View** | "What needs attention this week, and why?" | Salesforce, Pendo, Zendesk, news/studies, Chatter | Each CSM's own book |
| **Tasks** | "What's on the calendar - renewals, QBRs, reviews - for the whole team?" | Salesforce renewal dates, on a repeating touchpoint schedule (see [iteration notes](docs/iteration-notes.md)) | CSMs and managers planning ahead |
| **Heat Map** | "Show me every account, colored by urgency, with a full per-account detail panel." | All of the above, plus Clari and the report catalogs below | Anyone who needs the complete picture on one account |
| **CS Team Forecast** | "What does the org actually expect to close this month and quarter - one number, one definition, every team?" | Salesforce renewal + upsell opportunities, bucketed by one shared rule set applied identically to every CSM | Leadership and the whole CS/AM org |

Four tabs, not one, because stretching "what needs my attention," "what's on the calendar,"
"everything about this one account," and "the org-wide number leadership needs" into a single
view would mean compromising all four. Each one exists because a real person asking a real
question needed the answer shaped differently than the others - see
[docs/design-decisions.md](docs/design-decisions.md) for the tab-by-tab reasoning, most recently
[why the CS Team Forecast tab](docs/design-decisions.md#why-a-tab-4-cs-team-forecast-tab).

Clicking any account in any tab jumps to its detail panel - no separate lookup, no tab-switching
to piece the picture together by hand.

**Optional companions**, built the same no-code way, that the Heat Map's per-account panel can
cross-reference: a searchable catalog of **published studies that used your product** and a
searchable catalog of **reports done with competitor tools** - both good sources for "who else
in this space already sees the value" and "what are we losing deals to." Ask Claude to build
these the same way as the main dashboard if you want them; they're independent artifacts, not
required for the core routine.

---

## What changed in v2

Four things changed after running v1 for real, for months, against a full account book:

1. **Added Zendesk** (support tickets - volume, open count, category) once API access existed.
   No official Zendesk MCP connector exists, so - same as Salesforce - it's a short custom
   setup. See [docs/adding-a-custom-source.md](docs/adding-a-custom-source.md) for the general
   pattern, and [design-decisions.md](docs/design-decisions.md#why-add-zendesk-after-all) for
   why CSAT specifically did **not** make the cut.
2. **Added Clari** (forecast/pipeline context) as an optional signal, same custom-MCP pattern,
   strictly read-only - see [design-decisions.md](docs/design-decisions.md#why-clari-and-why-its-optional).
3. **Added the Heat Map tab** - a full per-account panel pulling from every connected source at
   once, instead of that detail only existing implicitly across several tools a person would
   have to check separately.
4. **Walked back signal-only filtering.** Every account in the book now gets reviewed every run;
   accounts with genuinely nothing to report land in a collapsed, neutral "Rest of Accounts"
   group instead of disappearing entirely. See
   [why, below](docs/design-decisions.md#why-we-walked-back-signal-only) - it's the most
   important lesson in this whole project.

---

## What changed in v3

**Added the CS Team Forecast tab** - a fourth tab for a different audience than the first three.
Not "what does one CSM need to act on this week," but "what does the whole team's renewal and
upsell forecast add up to, right now, on one definition everyone actually agrees on." It replaced
a spreadsheet-pooling process across roughly 15 CSMs and AMs that had drifted into what the team
called "hopecasting" - commit and upside figures merged into one bucket with no flag that they
weren't the same kind of number, producing a forecast that routinely missed reality in both
directions. See [design-decisions.md](docs/design-decisions.md#why-a-tab-4-cs-team-forecast-tab)
for the full reasoning, and [iteration-notes.md](docs/iteration-notes.md) for the correction
rounds it took to get the field-level definitions exactly right - this tab is the one place in
the whole dashboard where multiple people's numbers land in a single figure, so precision here
mattered more than anywhere else in the project.

---

## Results & Impact

This has been running as a real weekly routine - not a demo - since June 2026, against an
active, multi-account CS book, and has grown from four data sources to six over that time.

**What it replaced:** a manual weekly pass across every account, cross-referencing a CRM, a
support desk, a usage-analytics tool, a forecast tool, and outside news by hand, then writing up
a summary - repeated every week, for every account, regardless of whether anything had actually
changed.

**What changed:**

- **Multi-source correlation on autopilot, across a growing number of systems.** Every account
  gets renewal timing, usage trend, support-ticket load, forecast context, external news, and
  last-touch/handoff-note context assembled automatically each week - the kind of
  cross-referencing that quietly stops happening by hand the first busy week, and that gets
  harder to do by hand every time you add a source, not easier.
- **Caught things a manual process had been missing** - and kept catching new ones as sources
  were added. See [docs/iteration-notes.md](docs/iteration-notes.md) for specifics: a name
  change that had been silently dropping a manager's entire book from view; usage-risk false
  positives from accounts used indirectly through a partner or child account; a support desk
  where ticket-to-organization tagging turned out to be unreliable on ~90% of tickets, requiring
  a different matching approach entirely; and - the big one - a filtering design that was
  quietly hiding real usage risk on accounts it had decided weren't worth showing.
- **A fuller picture per account, not just a longer list of sources.** The Heat Map's
  per-account panel is the "every corner of every system, one screen" view - the thing that
  used to require opening four or five separate tools to reconstruct by hand.
- **Resilient by design.** When a data source is temporarily unreachable, the routine still runs
  and flags what it couldn't check, rather than failing outright or silently omitting it (see
  [docs/iteration-notes.md](docs/iteration-notes.md)).
- **One forecast number instead of ~15 people's spreadsheets.** The CS Team Forecast tab replaced
  a manual pooling process across the whole CS/AM org with one live, identically-defined number
  per CSM - closing the gap between what leadership expected and what actually happened that the
  team had started calling "hopecasting."

*(Figures and specifics above reflect this project's own production history. If you adapt this
for your own team, swap in your own measurements and sources - the structural lessons don't
depend on any particular numbers or tool list to be true.)*

---

## How it's built (plain version)

```
                    +-------------------------------------------------+
                    |             Claude (desktop app)                |
                    |       runs your routine once a week             |
                    +-------------------------------------------------+
                       |        |         |        |         |         |
                  read |   read |     read |   read |     web |   build/send
                       v        v         v        v         v         v
                 Salesforce  Pendo   Zendesk    Clari   Web search   Artifact
                   (MCP)   (connector) (MCP)     (MCP)                dashboard
                                                                     (4 tabs, stable URL)
                                                                          |
                                                                          v
                                                              Slack (connector) + optional
                                                              Tability check-in (connector)
```

Slack, Pendo, and Tability plug in through **Claude connectors** - secure links to tools your
company already uses, enabled with a click and a browser sign-in. **Salesforce, Zendesk, and
Clari are the exceptions:** none of them had a ready-made connector for this org, so each is
added with a short one-time **MCP setup** - the same pattern applied three times. See
[SETUP.md](SETUP.md) Step 3 and [docs/adding-a-custom-source.md](docs/adding-a-custom-source.md)
for the general recipe.

The dashboard artifact is built by Claude itself - no hosting, no server, no external service.
It's a web page Claude publishes to claude.ai. You link to it from Slack.

---

## Colored urgency indicators

Each account in the **360 Account View** and **Heat Map** gets a color-coded card based on
urgency:

- Red (Act Now) - renewal within 30 days, OR a high-value account with no activity in over 90
  days, OR a lost renewal still inside its close month (still recoverable through month-end)
- Blue (Watch) - renewal 31-90 days out, OR moderate usage risk (quiet for 60-90 days)
- Green (FYI) - informational signal only (news, published study, healthy usage); no immediate
  risk
- Gray (Rest of Accounts) - nothing to report this week; collapsed by default so the meaningful
  tiers stay in view, but still searchable/filterable and still reviewed every run

A lost renewal stays Act Now for the rest of its close month - it's still recoverable until then.
Once that month ends without recovery, the account is no longer an active customer: it's treated
as churned and drops off the book entirely, rather than lingering as an informational FYI card.

---

## Get started

[SETUP.md](SETUP.md) - the click-by-click guide (Windows and Mac).

In short:
1. Install the **Claude desktop app** and sign in.
2. **Connect** Slack and Pendo (and Tability if you use it) - one click each.
3. **Add Salesforce, Zendesk, and Clari** via the short MCP setup ([SETUP.md](SETUP.md), Step 3;
   pattern detailed in [docs/adding-a-custom-source.md](docs/adding-a-custom-source.md)) - or
   hand that step to a technical teammate or IT. Zendesk and Clari are optional; add them
   whenever you're ready, the routine works with just Salesforce + Pendo in the meantime.
4. Paste the routine from [examples/weekly-digest-task.md](examples/weekly-digest-task.md) and
   ask Claude to run it weekly.

---

## Read-only by design

This agent only ever **reads** your systems of record - it should never change data in
Salesforce, Zendesk, or Clari. The best way to guarantee that for Salesforce is a **read-only
login**; ask your admin to set one up using
[docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md). For Zendesk and
Clari, the custom MCP servers built in
[docs/adding-a-custom-source.md](docs/adding-a-custom-source.md) simply never expose a write
tool - there's nothing to accidentally call.

---

## What's in this repo

| Path | What |
|------|------|
| [SETUP.md](SETUP.md) | **Start here** - step-by-step setup (mostly click-based) |
| [docs/design-decisions.md](docs/design-decisions.md) | Why each choice was made, and what was rejected |
| [docs/iteration-notes.md](docs/iteration-notes.md) | What broke running this for real, and what's next |
| [docs/adding-a-custom-source.md](docs/adding-a-custom-source.md) | The reusable pattern for connecting a tool with no official MCP connector (used for Salesforce, Zendesk, and Clari) |
| [salesforce/salesforce-mcp-config.json](salesforce/salesforce-mcp-config.json) | Salesforce MCP config (read-only) - used in SETUP Step 3 |
| [zendesk/zendesk-mcp-config.json](zendesk/zendesk-mcp-config.json) | Zendesk MCP config (read-only, custom server) |
| [clari/clari-mcp-config.json](clari/clari-mcp-config.json) | Clari MCP config (read-only, custom server, optional) |
| [examples/weekly-digest-task.md](examples/weekly-digest-task.md) | The routine instructions you give Claude |
| [examples/accounts.sample.json](examples/accounts.sample.json) | **Fake** account list (shows the shape) |
| [docs/sample-slack-digest.md](docs/sample-slack-digest.md) | Example of the weekly Slack summary and dashboard |
| [docs/salesforce-readonly-user-setup.md](docs/salesforce-readonly-user-setup.md) | For your admin: read-only Salesforce login |
| [docs/google-docs-connector-request.md](docs/google-docs-connector-request.md) | Optional extension: what to ask IT for to enable auto-updating account docs |
| [advanced/](advanced/) | Optional: extra client-side write-protection guardrail |

---

## v1 recap

The original version of this project (four sources: Salesforce, Pendo, web search, Slack, plus
optional Tability; strict signal-only filtering; a single-tab dashboard) is preserved in this
repo's git history and in [docs/design-decisions.md](docs/design-decisions.md) /
[docs/iteration-notes.md](docs/iteration-notes.md), which keep both the v1 reasoning and the v2
changes side by side rather than overwriting the record of what was tried first.

---

## Privacy

This tutorial contains **no real data** - every name, account, ticket, and ID is a placeholder
or fake example. Your real customer list, tickets, forecasts, and credentials stay inside your
own Claude app and connected tools; nothing sensitive lives in this repo.

---

## License

MIT - use it, adapt it, share it with your team.
