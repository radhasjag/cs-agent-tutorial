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
instead of a name string would fix this at the root - on the v2 list below.

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
- **Fewer urgency tiers.** Started with four urgency levels, collapsed to three. The fourth tier
  didn't change what anyone did - it just added a judgment call about which bucket something
  belonged in.

---

## What's next (v2 candidates)

- Match ownership by a stable ID instead of a name string - closes issue #1 at the root instead
  of just detecting it after the fact.
- Add a support-ticket source (e.g., Zendesk) as a fourth signal type, once connected - see
  [design-decisions.md](design-decisions.md) for why it wasn't in v1.
- Auto-updating companion docs are blocked on a create-only connector limitation - revisit if an
  edit-capable one becomes available (see
  [google-docs-connector-request.md](google-docs-connector-request.md)).
- Make the optional OKR check-in reflect week-over-week movement between tiers, not just a
  snapshot.
