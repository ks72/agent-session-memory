# Worked example — raw summary to layered log

One session, shown twice. First the flat summary a typical agent produces at wrap-up, then what the layering rules turn it into.

The point of the comparison is not that the second is longer. It is that the second sorts facts by *what kind of thing they are*, so the next session knows which parts to trust.

---

## Before — what a flat summary looks like

> Today we worked on the Q1 webinar campaign for Meridian's security product. We fixed the lead-quality problem by adding a company-size question to the registration form. We also noticed MQL-to-SQL conversion has been dropping since last week, which was pulling down the numbers. We decided to route all webinar leads through a manual review before they hit Sales, instead of auto-qualifying on form fields alone. Sales prefers a weekly pipeline summary over daily Slack pings on every new lead. Next time we should check whether the LinkedIn ad set is still targeting the right job titles. Also the drop-off on the thank-you page seems to be a tracking issue.

Everything true, nothing sorted. Four problems, and all of them cost something later:

1. **"MQL-to-SQL conversion has been dropping since last week"** — presented as established fact. It came from one pull of the CRM report, on one campaign. It might be a real quality problem, a sales-side capacity issue, or a reporting lag. Written like this, the next session treats it as settled and stops asking.
2. **"the drop-off on the thank-you page seems to be a tracking issue"** — the word "seems" is the only marker that this is a guess, and it will not survive the next summarization. Two sessions from now it reads as fact.
3. **"Sales prefers a weekly pipeline summary over daily Slack pings"** — a durable working preference, buried in a dated log where nobody will look for it.
4. **"check whether the LinkedIn ad set is still targeting the right job titles"** — an open item with no horizon. Is it urgent or someday? Unanswerable from this text.

---

## After — sorted by layer

### `.session-log/meridian-q1-webinar/2026-01-15.md` (the dated log)

```markdown
# Session Log — 2026-01-15 · meridian-q1-webinar

## What we did
Fixed the lead-quality problem by adding a company-size question to the
registration form. Set a manual review step before webinar leads reach
Sales, instead of auto-qualifying on form fields alone. Noticed
MQL-to-SQL conversion dropping since last week, though the cause is not
yet established.

## Decisions made
- Manual review before Sales handoff, rather than auto-qualifying on form
  fields — the company-size field alone wasn't filtering out job seekers
  and students registering for the free resource.
- Weekly pipeline summary instead of daily Slack pings — matches Sales's
  stated preference.

## Open items
- [ ] Check whether the LinkedIn ad set is still targeting the right job
      titles

## Discoveries
- Registration form had no company-size or seniority field at all; roughly
  a third of leads from the last campaign were individual contributors at
  companies too small to be in the target segment.

## Assumptions to validate
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 1×, 0 counter-examples
- Thank-you page drop-off is a tracking/pixel issue, not users actually
  leaving — seen 1×, 0 counter-examples

## Files produced
- `campaigns/webinar-q1-form-v2.md` — updated registration form fields and
  routing rules
```

### `.session-log/meridian-q1-webinar/context.md` (running state — updated, not appended)

```markdown
## Open items

### Active
- [ ] Check whether the LinkedIn ad set is still targeting the right job
      titles

### Backlog
- [ ] Revisit the manual-review step once a full cycle of data exists

### Someday / Blocked
- [ ] Add lead scoring by intent signal — blocked on marketing ops
      finishing the tracking-pixel audit

## Assumptions to validate
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 1×, last 2026-01-15, 0 counter-examples
  (3 checks, same CRM report, same session — not independent, see note)
- Thank-you page drop-off is a tracking/pixel issue, not users actually
  leaving — seen 1×, last 2026-01-15, 0 counter-examples
```

Note the correction on the first assumption. During the session it was checked three times, which looks like a tally of 3. All three checks came from re-running the same CRM report, in the same session, on the same day — nothing varied, so nothing could have disagreed. The honest tally is 1.

That correction is the whole mechanism working. An agent that bumped it to 3 would have promoted "lead-quality issue" to a rule on evidence that never tested the *lead-quality* part of the claim — it never checked whether Sales's own response time had changed.

### Durable memory (L1)

```markdown
Meridian's Sales team prefers a weekly pipeline summary over daily Slack
pings on every new lead — batch updates, don't ping as things happen.
```

This is a working preference that will be true in six months and applies beyond this one task. It does not belong in a dated log, and it does not belong in `context.md`, which gets pruned. It goes in the layer that survives.

---

## What each rule bought

| Rule | What it prevented |
|---|---|
| Assumptions get their own section and a tally | Two guesses being read as fact by the next session |
| Independence check before bumping a tally | A claim about a *lead-quality* problem graduating on evidence that never checked Sales's own capacity |
| Open items tiered by horizon | An unranked list where a blocked item and an active one look identical |
| Durable preferences go to L1 | A standing Sales-side preference decaying inside a dated log |
| The dated log records, `context.md` owns | Two copies of the open-items list drifting apart |

---

## Three sessions later

The conversion-drop assumption gets tested properly — Sales's own weekly capacity report, checked on a different day:

```markdown
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 2×, last 2026-01-22, 0 counter-examples
```

Then Sales's capacity report shows two reps out on leave the same week the drop started:

```markdown
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 0×, last 2026-01-24, 1 counter-example
  (two Sales reps were out the same week conversion dropped — suggests
  capacity, not lead quality)
- MQL-to-SQL conversion drop is a Sales capacity issue, tied to reduced
  headcount that week — seen 1×, last 2026-01-24, 0 counter-examples
```

One counter-example reset a tally of 2 to zero, and the narrower claim started its own count at 1. Had the original been promoted to a rule at the first session's apparent "3×", every subsequent session would have been reasoning about campaign performance from a wrong model — and nothing in the record would have flagged it.
