# Graduation examples — worked cases

Two complete walkthroughs of the graduation rule from `SKILL.md`: one assumption that earns promotion to a lesson, one that looks identical on paper and does not.

Read this when applying the rule to a real assumption for the first time, or when a tally reads "3×" but something feels off about it.

## Contents

- [The rule in one line](#the-rule-in-one-line)
- [Case 1 — graduates](#case-1--graduates)
- [Case 2 — does NOT graduate](#case-2--does-not-graduate)
- [Telling the two apart](#telling-the-two-apart)
- [What contradiction does](#what-contradiction-does)

---

## The rule in one line

An assumption graduates into a lesson after **3 independent corroborations with zero counter-examples**.

> Two corroborations are **independent** when they could have disagreed. Different session, different trigger, different surrounding conditions — such that the assumption being wrong would plausibly have shown up in at least one of them.

Everything below is that definition applied twice.

---

## Case 1 — graduates

Starting state in `context.md`:

```markdown
## Assumptions to validate
- Signup drop-off is caused by the new pricing page, not the checkout
  flow — seen 1×, last 2026-01-03, 0 counter-examples
```

**Session 1 — Jan 3.** Signups fall the week the new pricing page goes live. Two explanations fit: the pricing page itself, or something unrelated in checkout that shipped the same week. No way to tell yet. Recorded as an assumption, tally 1 — deliberately *not* as a discovery, because nothing has been proven.

**Session 2 — Jan 9.** Unrelated work on a different campaign. A/B test data comes back: the old pricing page, run as a holdout for a separate experiment, keeps its normal signup rate while the new page underperforms. Nobody was looking for this; it surfaced on its own.

Independent? Yes. Different task, different trigger, and it isolates the variable directly. If the cause had been the checkout flow, the holdout with the old pricing page would have dropped too — it did not. Tally 2.

```markdown
- Signup drop-off is caused by the new pricing page, not the checkout
  flow — seen 2×, last 2026-01-09, 0 counter-examples
```

**Session 3 — Jan 15.** Someone reverts the pricing page copy to the old version while leaving checkout untouched. Signups recover within two days.

Independent? Yes, and by a different mechanism — this is an intervention, not an observation. If the cause had been checkout, reverting the pricing page would have changed nothing. Tally 3, zero counter-examples.

**→ Graduates.** Three occasions, each of which could have contradicted the assumption, none of which did. Promote it to `LESSONS.md`:

```markdown
### SL-004 — Pricing page copy drove the signup drop, not checkout
**Date:** 2026-01-15
**Context:** Signup rate fell after the Q1 pricing page redesign
**What failed:** Chasing checkout-flow issues across two sessions
**Fix:** Reverted the pricing page copy; signups recovered within two days
**Prevention rule:** Treat a signup drop that starts the same week as a
pricing/positioning change as copy-related first — check the holdout or
revert before investigating the checkout flow.
```

Then **delete it from `context.md`**. It now lives in exactly one layer.

---

## Case 2 — does NOT graduate

Starting state:

```markdown
## Assumptions to validate
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 1×, last 2026-01-15, 0 counter-examples
```

**Session 1, 14:02.** The CRM report is pulled. Conversion is down. The assumption forms: lead quality. Tally 1.

**Session 1, 14:20.** The same report, re-run to double-check. Conversion still down. Tally bumped to 2.

**Session 1, 14:41.** Filtered by campaign, same report. Still down. Tally bumped to 3.

```markdown
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 3×, last 2026-01-15, 0 counter-examples
```

On paper this is identical to Case 1: three corroborations, zero counter-examples, threshold met.

**→ Does NOT graduate.**

All three observations came from one report, one session, one underlying number viewed three different ways. Nothing varied between them, so nothing could have disagreed. Re-reading the same drop three times does not test the *lead-quality* part of the claim even once — it only confirms that conversion is down, which was never in question.

The corrected entry:

```markdown
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 1×, last 2026-01-15, 0 counter-examples
  (3 views of one report, same session — not independent)
```

Confirming this properly needs variation on the dimension the claim is *about*: lead scores compared before and after the drop (are the leads actually worse?), Sales's own capacity that week (were reps unavailable?), or the same lead volume routed to a different rep (does conversion change with who's working it?).

---

## Telling the two apart

The useful question is not "how many times have I seen this?" but:

> **What else would explain what I saw?**

In Case 1, each new observation ruled something out. The holdout page keeping its normal rate argued against checkout. The revert fixing it and nothing else changing argued against anything else shipped that week. The set of surviving explanations kept shrinking.

In Case 2, lead quality, Sales capacity, and reporting lag all explain all three observations equally well after the third view as after the first. The number went up; the uncertainty did not go down.

**A tally that grows without narrowing the explanation space is measuring persistence, not truth.**

Practical checks before bumping a tally:

| Ask | If the answer is no |
|---|---|
| Did anything vary between this observation and the last? | Do not bump. Annotate instead. |
| Could this observation have come out the other way? | Do not bump. It was not a test. |
| Does this rule out an explanation the previous ones allowed? | Bump cautiously; note what it ruled out. |
| Am I the one who went looking for it? | Bump, but weight an accidental confirmation higher — nobody was steering it. |

---

## What contradiction does

One counter-example resets the tally to 0. Not to 2, not "3 minus 1" — to zero.

This is deliberately harsh, and it is the mechanism's whole point. An assumption at tally 3 is one step from being written down as a rule that every future session inherits without re-checking. If reality just contradicted it, the accumulated confidence was measuring something other than truth, and the count that produced it should not be trusted to survive.

If the counter-example shows the assumption is simply wrong, delete it rather than resetting.

If it reveals the assumption was *nearly* right but wrongly scoped — the drop was real but tied to one campaign, not the pricing page site-wide — rewrite it as the narrower claim and start its tally at 1. It is a new assumption; it has not been independently corroborated three times yet.
