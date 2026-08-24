# Sample Digest Output (fake data)

What the weekly output looks like. All accounts, people, and numbers below are **made up**.

---

## Dashboard artifact (full detail, three tabs)

The dashboard redeploys to the same Claude artifact URL every week - "360 Account View" (every
account, tiered), "Tasks" (renewals/QBRs due, by month), and "Heat Map" (full-book color grid
with a per-account panel pulling from every connected source). See
[README.md](../README.md#three-views-one-dashboard) for what each tab covers.

---

## Slack DM (top 3 only)

What lands in your Slack DM each Thursday:

---

*Weekly Digest - Thu, Jun 26* - 4 renewals within 90d - 2 usage risks - 91 accounts reviewed

Red circle *Globex Energy* - CS: Jordan Lee - AM: Dana Kim
  Renews Jul 2 (6d) - $240,000 - usage: RISK - no visit 94d (last Apr 2026)
  Tickets: 12 this year, 3 open, mostly billing
  News: Announced $500M grid expansion in Texas - example.com - angle: Infrastructure spend = economic impact use case
  Last touch: Jordan Lee, Jun 20 - Renewal discussion

Blue circle *Acme Corporation* - CS: Jordan Lee - AM: Dana Kim
  Renews Aug 15 (50d) - $180,000 - usage: active ~3d ago
  Tickets: 2 this year, 0 open
  News: New manufacturing plant in Ohio - example.com - angle: New facility = natural impact analysis project
  Last touch: Jordan Lee, Jun 9 - Quarterly check-in

Blue circle *State University of Example* - CS: Sam Patel - AM: Pat Rivera
  Renews Sep 1 (67d) - $95,000
  Study: "Economic Impact of University Research Grants" used our product - example.edu
  Handoff note: "Heads up, new department contact as of May - loop in before the renewal call." (Pat Rivera, Jun 2)
  Last touch: Sam Patel, May 28 - Onboarding follow-up

Full dashboard (91 accounts, 4 Act Now / 11 Watch / 22 FYI / 54 Rest of Accounts): https://claude.ai/code/artifact/EXAMPLE_ARTIFACT_ID

---

## Circle key

- Red circle (ACT NOW) - renewal within 30 days, OR high-ARR account dormant >90 days, OR a lost
  renewal whose due date hasn't passed yet
- Blue circle (WATCH) - renewal 31-90 days out, OR moderate usage risk (quiet 60-90 days)
- Green circle (FYI) - informational only (news, published study, healthy usage, a lost renewal
  past its due date); no immediate risk
- Gray (REST OF ACCOUNTS) - nothing to report this week; still reviewed, just collapsed by
  default in the dashboard so it doesn't crowd out the tiers that need attention

---

*If no account across the whole book has any signal, the Slack message is simply:*
> All quiet this week - no renewals within 90 days, usage risks, or notable news. 91/91 accounts
> reviewed, all in Rest of Accounts.

*If a connected source is down, it's called out explicitly rather than silently skipped:*
> Note: Zendesk was unreachable this run - ticket data omitted, everything else current.
