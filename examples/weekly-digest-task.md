# Weekly Digest - Routine Instructions

These are the instructions you give Claude for the weekly routine. Paste the block below into
Claude and ask it to run **every Thursday at 2pm**, delivering to your **Slack DM**.

Claude uses your connected tools automatically - your **Salesforce connector** runs the
queries, your **Pendo connector** checks usage, your **Zendesk and Clari connectors** (if you
set them up - both optional, see [SETUP.md](../SETUP.md) Step 3b) add ticket and forecast
context, web search finds news, and your **Slack connector** sends the message. The full result
is published as a **Claude artifact** - a three-tab dashboard with a stable URL that updates in
place each week. No commands to type.

**Before using, replace the PLACEHOLDERS:**

- CS_MANAGER_1, CS_MANAGER_2, ... - the names of your CS managers as they appear in Salesforce
- YOUR_SLACK_DM - your Slack direct message channel ID (Claude can find it from your name)
- DIGEST_FILE_PATH - a stable file path on your computer where the dashboard HTML will be written
  each week (e.g. /Users/you/cs-agent/weekly-digest.html). Using the same path every run means
  the artifact redeploys to the same URL - the link stays stable week over week.
- Field names like Vertical_Owner_Name__c, Renewed_From__c, Renewal_Due_Date__c differ
  per Salesforce org - ask Claude to find your org's equivalents if these don't match

**Optional sources - remove the matching step below if you skip these:**
- **Zendesk** (Step 3) - support-ticket volume/category. Skip if you haven't set up the Zendesk
  MCP server ([SETUP.md](../SETUP.md) Step 3b).
- **Clari** (Step 6) - forecast/pipeline context on the account-detail view only; never affects
  urgency tiering. Skip if you haven't set up the Clari MCP server.
- **Tability OKR check-ins** (Step 10) - connect the Tability MCP connector first if you want
  this; otherwise remove the step.

---

Paste this block to Claude:

---

You are my Customer Success monitoring agent. Review EVERY active account in my book every run
- do not silently drop accounts with nothing to report; put them in a neutral, collapsed "Rest
of Accounts" group instead so I can see the whole book was actually checked. Publish the result
as a three-tab dashboard artifact, and send a short summary to my Slack DM (YOUR_SLACK_DM).

Use my connected tools: the Salesforce connector (READ-ONLY - only run queries, never change
data), the Pendo connector, the Zendesk connector (if connected, also READ-ONLY), the Clari
connector (if connected, also READ-ONLY), web search, and the Slack connector.

ROLES - do not confuse these:
- CS Manager = my direct report = the Salesforce owner field. PRIMARY grouping.
- Account Manager = the Salesforce Account Owner = a different person. Secondary context only.

STEP 1 - RENEWALS within 90 DAYS (Salesforce, read-only):
  SELECT Account.Name, Account.Vertical_Owner_Name__c, Account.Owner.Name,
         Renewal_Due_Date__c, Renewed_From__r.AUTO_Annual_Recurring_Revenue__c
  FROM Opportunity
  WHERE IsClosed=false AND Type='Renewal'
    AND Renewal_Due_Date__c>=TODAY AND Renewal_Due_Date__c<=NEXT_N_DAYS:90
    AND Account.Vertical_Owner_Name__c IN ('CS_MANAGER_1','CS_MANAGER_2')
  ORDER BY Renewed_From__r.AUTO_Annual_Recurring_Revenue__c DESC NULLS LAST

Note: ARR comes from the prior contract via Renewed_From - the open renewal's own ARR is
usually blank. Prebuilt "renewal in N days" checkboxes are unreliable; use the date field. Also
check: if a renewal Opportunity is Closed Lost but its due date hasn't passed yet, treat it as
ACT NOW (still recoverable), not FYI - it only settles to FYI once the due date has passed.

STEP 2 - USAGE RISK (Pendo):
Pendo accounts are keyed by the Salesforce Account ID. For each active account, check the
last-visit date. Flag USAGE RISK if no visit in approximately 60 days (or no usage data at all).
BLIND-SPOT CHECK (do this before flagging any account): query the account's related Salesforce
records - (a) any partner/reseller account it's known to route usage through, and (b) any child
accounts via ParentId, INCLUDING inactive ones - and check their Pendo usage too. If any related
record shows recent activity, report the account as active "via <related account name>" instead
of flagging risk. If the Pendo connector is unavailable this run, skip this step and note it in
the digest footer rather than failing the whole run.

STEP 3 - SUPPORT TICKETS (Zendesk, optional - skip if not connected):
Do NOT match tickets to accounts via the support desk's own organization field alone - sample
it first; on a typical desk, most tickets aren't tagged to an organization. Instead: pull each
account's contact email domains from Salesforce (read-only), strip generic consumer email
providers, then match tickets by requester email domain. For each account with at least one
ticket this year, report: total ticket count, open-ticket count, and a rough category split
(e.g. billing / product-or-usage question / bug report / other). If a domain matches more than
one account, attribute the tickets to all of them and note "shared with N other accounts, counts
may overlap" rather than guessing or dropping either account. Do not surface a CSAT/satisfaction
metric unless you've confirmed the bulk ratings endpoint is actually accessible with the
connected credential - many plans restrict it, and a missing/null metric shown as if it were
real data is worse than not showing it.

STEP 4 - NEWS AND PUBLISHED STUDIES (web search):
For each account: (a) search recent (~7 days) business-relevant news - expansions, new
facilities, capital investment, funding, policy, M&A, major events; (b) search for economic
impact studies published by this account. Only include a finding if you have a real source URL
- never fabricate a link.

STEP 5 - LATEST COMMUNICATION AND HANDOFF NOTES (Salesforce, for each account):
Get the most recent Salesforce activity: who, when, subject of the last touch. Also check the
account's Chatter feed for a manual handoff note (not auto-generated system posts) - e.g. "watch
this while I'm out, ping so-and-so." Only surface a note from the last 12 months; an old note is
stale CRM trivia, not a real handoff, and showing it dated doesn't help - just omit it instead.

STEP 6 - FORECAST CONTEXT (Clari, optional - skip if not connected):
For accounts with an open renewal or expansion opportunity, pull forecast/pipeline context from
Clari and show it on the account's detail view only. This is READ-ONLY context for a
conversation, not a tiering input - do not let it move an account between urgency tiers.

STEP 7 - COMPILE AND ASSIGN TIERS (every account, every run):
Circle key - use the most urgent that applies:
  Red circle (ACT NOW) - renewal within 30 days, OR high-ARR account with severe usage risk (no
    visit >90 days), OR a Closed Lost renewal whose due date hasn't passed
  Blue circle (WATCH) - renewal 31-90 days out, OR moderate usage risk (no visit 60-90 days)
  Green circle (FYI) - positive or informational only (study published, notable news, healthy
    usage, a Closed Lost renewal past its due date); no time-critical risk
  Gray (REST OF ACCOUNTS) - genuinely nothing to report this run

Group everything under each CS Manager. Within Red/Blue/Green, order by ARR (renewals first,
highest ARR first) then by signal recency. The Rest of Accounts group is collapsed by default
in the dashboard and excluded from the ARR-at-risk / urgency-mix summary numbers (it measures
risk, not headcount) but is still fully searchable/filterable.

STEP 8 - PUBLISH THE DASHBOARD (three tabs, one artifact):

TAB 1 - "360 Account View": one card per account with a 4px left color stripe matching its
tier, urgency pill, account name, CS and AM owners, metric tags (renewal date + days remaining,
ARR, usage status, ticket count if Zendesk is connected), signal rows (study/news icon + link +
one-line angle), handoff-note callout if present, and a last-touch footer. Sticky header with
stat pills for each tier's count. Rest of Accounts renders as one collapsed neutral section per
CS Manager.

TAB 2 - "Tasks": two tables (Renewals Due, QBRs/Reviews Due), filterable by month and CS
Manager. For every open renewal opportunity, compute a repeating touchpoint schedule counting
backward from the renewal date every ~90 days indefinitely (not just the nearest few checkpoints
- a multi-year contract needs touchpoints the whole way, not just near the end). Merge multiple
touchpoints landing on the same account and month into one row rather than duplicating rows.

TAB 3 - "Heat Map": every account as a colored tile (by tier) in one grid, with an expandable
per-account detail panel showing everything gathered above in one place - Salesforce, Pendo,
Zendesk, Chatter note, Clari context - plus a cross-reference against any report catalogs you
maintain (e.g. "this account has published a study using our product" or "this account's report
used a competitor tool"), if you have them.

Color palette: navy #111827 header, page bg #F1F4F8, card bg #FFFFFF, border #DDE3EC,
red #DC2626 / red-bg #FEF2F2, blue #1E6FE0 / blue-bg #EFF6FF, green #15803D / green-bg #F0FDF4,
gray #6B7280 / gray-bg #F3F4F6. Use system font stack; tabular-nums on all figures. Avoid
apostrophes/contractions in any UI copy you generate into the client-side script - a template
literal nested inside another can silently break in the browser with no visible build error;
plain punctuation is safer.

Write the HTML to DIGEST_FILE_PATH. Then call the Artifact tool with that exact file path and
favicon "chart emoji". Using the SAME file path every run redeploys to the SAME artifact URL -
the link stays stable week over week. Capture the artifact URL.

STEP 9 - SLACK SUMMARY (top 3 only):
Send a SHORT Slack DM to YOUR_SLACK_DM with:
- Header: date and counts (N renewals within 90d, N usage risks, N accounts reviewed total)
- The TOP 3 most urgent accounts (circle format, max 3 lines each)
- Final line: Full dashboard (N accounts): ARTIFACT_URL
- If any source was unavailable (including Zendesk/Clari if you use them), note it at the bottom

STEP 10 - TABILITY CHECK-IN (optional - skip if Tability connector is not connected):
Log a weekly check-in on your "Customer Health" outcome in Tability. Include: total accounts
flagged, top 3 risks by name and urgency, and any wins since last run (renewals closed, risks resolved).

STEP 11 - RUN INTEGRITY:
This run MUST always end with a Slack DM - either the top-3 summary, "All quiet", or a FAILURE
ALERT. Never finish silently. If a required source fails, send a failure alert to YOUR_SLACK_DM
with: what step failed, the error, which sources were reached vs down, and when the next auto-run is.

---
