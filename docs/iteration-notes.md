# Iteration Notes

What actually broke while this ran for real, what changed as a result, and what's still on the
list. Kept here instead of buried in commit history because the failure modes are the useful
part - most of them would bite anyone building something similar.

---

## What broke, and the fix

**1. A rename silently dropped an entire manager's book of accounts.**
Ownership was matched against a hardcoded name string. When a CS manager's name changed in the
CRM, the match quietly failed and every account they owned vanished from the digest - no error,
just an empty result that looked like "no signal this week." Fix: treat a manager showing zero
accounts as a red flag to check for a rename, not as a clean book. Matching on a stable ID
instead of a name string would fix this at the root - still on the v3 list below.

**2. A data source going quiet was invisible.** One connector can silently expire its
authorization. When it did, the usage-risk check just went missing from the output with no
explanation - indistinguishable from "no usage risk this week." Fix: the routine now detects when
that source is unreachable, skips only the usage-risk piece, and says so explicitly, instead of
producing a quietly incomplete result.

**3. False "gone quiet" flags from indirect usage.** Some accounts are used through a reseller or
a related child account rather than directly, so the account itself looks dormant in the usage
tool while real usage is happening elsewhere. This produced false churn-risk flags on accounts
that were, in fact, fine. Fix: before flagging usage risk, also check known related accounts and
report through whichever one is actually active.

**4. A wrong field mapping silently misattributed a relationship owner.** An "owner" field had
been pulling from the wrong underlying field for a while, so the wrong name showed up
consistently. Fix: corrected the mapping, and added a rule that an empty source field stays
blank on the card - never fall back to a different field and risk showing a fabricated name.

**5. The touchpoint schedule only worked for near-term renewals.** The first version computed a
fixed number of checkpoints counting back from the renewal date, so a multi-year contract got no
touchpoints at all for years at a stretch. Fix: rebuilt as a repeating offset pattern (every N
days, indefinitely, counting back from renewal) so any contract length gets an evenly-spaced
schedule instead of running out.

**6. The dashboard link moved every week.** Without an explicit fixed target, each run published
to a new address and the previously-shared link went stale. Fix: pin the destination explicitly
on every run rather than assuming "same content" implies "same place."

---

## What changed from real usage (not bugs, just wrong calls)

- **Delivery split.** V1 sent the full multi-account digest straight into Slack. In practice that
  was too much to read in a chat thread. Split into: a full dashboard (for anyone who wants the
  detail) + a short Slack ping with only the most urgent items and a link to the rest.
- **Fewer urgency tiers, then a different fourth tier came back for a different reason.**
  Started with four urgency levels, collapsed to three because the fourth didn't change what
  anyone did - it just added a judgment call about which bucket something belonged in. A fourth
  tier came back in v2, but as a neutral "Rest of Accounts" bucket for genuinely-quiet accounts,
  not as a fourth urgency level - a different problem than the one that got collapsed away the
  first time. See [design-decisions.md](design-decisions.md#why-we-walked-back-signal-only).

---

## What shipped in v2, and what broke getting there

**7. Support-desk tickets weren't tagged to an organization on ~90% of tickets.** The first
Zendesk integration matched tickets to accounts via the desk's own organization field - clean
in theory, since that's exactly the field meant for this. A live sample of 100+ recent tickets
found barely 1 in 10 actually had that field populated; most ticket volume came in through
channels (web widget, in-app chat) that never auto-stamp it. Org-based matching wasn't just
imperfect, it was silently undercounting nearly everyone. Fix: rebuilt matching on **requester
email domain**, derived from each account's own Salesforce contacts rather than an account-level
website field (which was itself missing on a meaningful slice of accounts). Domain-based
matching covered the whole book with no gaps.

**8. A handful of email domains legitimately belong to more than one account** (large
organizations - agencies, umbrella orgs - with several accounts sharing one domain). The
original plan was to skip any shared domain past a threshold, to avoid misattributing tickets.
Revisited: skipping loses real signal for every account on that domain, not just the ambiguous
one. Fix: attribute the ticket to *every* matching account and show a "shared with N other
accounts, counts may overlap" caveat, rather than silently dropping the account from view -
consistent with the broader lesson in item 9 below.

**9. Signal-only filtering was hiding real risk, not just noise.** A later usage-data check
found more than a dozen accounts with a genuine usage-risk signal that had simply never
appeared in any digest - not because anything was broken, but because the filter had been doing
exactly its job and that job was silently excluding real risk along with the noise it was meant
to cut. This is the most consequential finding in this project's whole run and is written up in
full in [design-decisions.md](design-decisions.md#why-we-walked-back-signal-only) rather than
here, since it's a design reversal, not a bug fix.

**10. A bulk satisfaction-rating (CSAT) endpoint returned a permissions error** with the API
credential available, and the ticket `priority` field was essentially unset on the vast
majority of tickets anyway. Rather than build a UI element for data that was either
unauthorized or mostly blank, CSAT was dropped from the ticket section entirely - see
[design-decisions.md](design-decisions.md#why-add-zendesk-after-all).

**11. A client-side template-literal escaping bug silently blanked the Heat Map tab.** The Heat
Map's browser-side script is generated as one large template literal inside the Node build
script. An escaped quote written for the *browser's* string parsing was being collapsed one
layer too early by *Node's own* template-literal parsing before it ever reached the browser -
producing a syntax error in the browser console with zero visible symptom in the build step
itself; the tab just rendered empty with no error banner. Fix: avoid the character entirely in
generated UI strings rather than fight the double-escaping. Lesson generalizes past this
project: **any time a build script generates code-as-a-string for a different runtime to parse,
open the actual output and check its console - a clean build log proves the generator ran, not
that what it generated is valid.**

**12. A usage-risk check missed a customer's real activity because it happened under a
different account record than the one being checked** - the same shape as the child-account
blind spot in item 3, but discovered again via Salesforce's native parent/child account
relationship rather than a separate partner organization, and notably the child record was
marked *inactive*, so a naive "active accounts only" query wouldn't have surfaced it at all even
if someone had thought to check. Folded into the existing partner/child usage check (item 3):
before flagging any account, also check its Salesforce children - even the inactive ones.

---

## What shipped in v3, and what broke getting there

The CS Team Forecast tab (see [design-decisions.md](design-decisions.md#why-a-tab-4-cs-team-forecast-tab))
went through more correction rounds against a reference than any other single piece of this
project - deliberately, since it's the one tab where several people's numbers land in a single
figure and "roughly right" isn't good enough for a number leadership will act on.

**13. A "won" check based on the wrong field silently under-counted closed deals.** The forecast's
closed-deal bucket was built on a boolean "won" flag. Some genuinely closed-and-paid deals didn't
have that flag set, because a separate in-between stage also counted as closed for this business
without tripping the flag. Fix: check the actual stage name directly instead of a derived
flag that didn't cover every stage that counted as closed.

**14. A pipeline figure came out roughly 40x too high because it was scoped to the wrong time
window.** One bucket was meant to be a single month's figure but was accidentally pulling in a
full quarter's worth of pipeline. Fix: scope every bucket to the same explicit month, consistently,
rather than leaving the window implicit and assuming it matched elsewhere in the sheet.

**15. Dead records still counted toward the forecast if they carried an old tag.** Duplicate and
closed-lost records were still being tallied into totals whenever they'd been tagged before being
marked dead, quietly inflating at least one bucket by a large amount. Fix: exclude dead and
duplicate records from every bucket outright, regardless of any tag left over from before they
were marked that way.

**16. The dollar-value field itself went through two corrections.** The first pass used a legacy
proxy value for open deals; a stakeholder review found it materially underquoted a real deal
against that deal's own record, so it moved to the deal's stated amount. A second review found
the business already had a dedicated "expected value" field, distinct from the general amount
field and closer to what "forecast" actually meant here - switching to it roughly tripled the
table's match rate against the original reference. Lesson, consistent with item 7 above: when a
business already has a specific field for the concept being computed, use that field - don't
assume a general-purpose column means the same thing everyone actually means.

---

## What's next (v4 candidates)

- Match ownership by a stable ID instead of a name string - closes issue #1 at the root instead
  of just detecting it after the fact. Still open; each rename gets caught and fixed
  individually rather than being structurally impossible.
- Fold Clari's forecast signal into urgency tiering, instead of showing it only as account-detail
  context - would need a clear rule for what forecast movement should actually change a tier,
  which doesn't have an obvious answer yet.
- Auto-updating companion docs are blocked on a create-only connector limitation - revisit if an
  edit-capable one becomes available (see
  [google-docs-connector-request.md](google-docs-connector-request.md)).
- Make the optional OKR check-in reflect week-over-week movement between tiers, not just a
  snapshot.
- The support-ticket category split still rides on a generic AI-generated tagging field not
  tuned to this business, so it's a rough proxy rather than ground truth - worth revisiting with
  a purpose-built classifier if the category breakdown starts driving real decisions.
