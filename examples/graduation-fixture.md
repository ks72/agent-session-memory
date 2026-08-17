# Behavioral fixture — does contradiction actually reset the tally?

Installing the skill proves files get written. It does not prove the graduation rule *behaves* as documented. This fixture tests the behavior.

Run it after installing, or any time you change the graduation section, to confirm the rule still works.

## What it tests

The claim under test: **one counter-example resets an assumption's tally to zero, rather than being absorbed or silently ignored.**

That is the mechanism's whole value. If a contradicted assumption keeps its count, a wrong belief can still graduate into a rule.

## Setup

Create `.session-log/meridian-q1-webinar/context.md`:

```markdown
# meridian-q1-webinar — running state

**Recognize this context by:** webinar, MQL, SQL, lead quality, Sales handoff

## Current state
Q1 webinar campaign for the security product. Manual lead-review step
landed last week.

## Open items

### Active
- [ ] Check whether the LinkedIn ad set is still targeting the right job
      titles

### Backlog

### Someday / Blocked

## Assumptions to validate
- MQL-to-SQL conversion drop is a lead-quality issue, not a Sales
  capacity issue — seen 2×, last 2026-01-22, 0 counter-examples
- Thank-you page drop-off is a tracking/pixel issue, not users actually
  leaving — seen 1×, last 2026-01-15, 0 counter-examples

## Decisions
- Manual review before Sales handoff, rather than auto-qualifying on form
  fields alone.
```

Note the first assumption sits at **2×** — one short of graduating.

## The session to feed it

Ask the agent to wrap up, giving it this as the session's events:

> - Continued work on the webinar lead-review process.
> - Pulled Sales's own weekly capacity report and found two reps were out on leave during the exact week MQL-to-SQL conversion dropped. Same week, same team, an entirely different report than the one used before.
> - Separately, the thank-you page drop-off happened again, on a different campaign landing page, built on a different template, during unrelated work.
> - I raised the lead-review SLA from same-day to next-business-day.

Two assumptions are deliberately pushed in opposite directions by the same session.

## Pass criteria

**1. The lead-quality assumption resets.** Two Sales reps out during the exact week conversion dropped is a counter-example: reduced capacity explains the dip at least as well as lead quality does. Expect the tally dropped to 0 or 1 with the reason recorded. A run that bumps it to 3× and graduates it is a **failure** — that is the exact error the mechanism exists to prevent.

**2. The tracking-issue assumption bumps to 2×, and does NOT graduate.** A different landing page template during unrelated work is genuinely independent, so the count should rise. But 2 is below the threshold, so it must stay in `## Assumptions to validate` rather than being promoted to `LESSONS.md`.

**3. Files land in the right place.** `.session-log/meridian-q1-webinar/` gains a dated log, and `context.md` is updated in place — not duplicated, and nothing written inside the skill's own directory.

Criterion 2 matters as much as 1. An agent that resets *everything* on any ambiguity is as broken as one that never resets — it would just never accumulate confidence at all.

## Recorded result

Run 2026-01-24 (Claude Code, agent given only `SKILL.md` and the fixture — no prior knowledge of this system):

| Criterion | Result |
|---|---|
| Lead-quality assumption resets | **Pass** — `seen 2×` → `seen 1×`, with the reason recorded inline |
| Tracking-issue assumption bumps, does not graduate | **Pass** — `seen 1×` → `seen 2×`, stayed in Assumptions |
| Files in the right place | **Pass** — dated log + updated `context.md` under `.session-log/meridian-q1-webinar/`, skill directory untouched |

The agent's own stated reasoning for the reset:

> two Sales reps out during the exact week conversion dropped is exactly the kind of observation that could have disagreed with "lead-quality issue" — and it does, because reduced capacity on the Sales side explains a conversion dip just as well as worse leads coming in.

That reasoning was produced from `SKILL.md` alone, which is the point: the rule has to work for an agent that has never seen this system before.

## Cross-agent check

The same independence rule was tested separately in Codex (v0.144.6), from a different starting point: session notes stating the thank-you page drop-off was seen **"three times"**, with no pre-existing tally to inherit.

Codex recorded it as `seen 1×`, and wrote its own justification into the file:

> Evidence so far is three views of the same landing page in one sitting; vary the page template, campaign, or day before treating this as independent corroboration.

Two different agents, two different framings, same correct call: three observations that could not have disagreed count as one. That the rule survives a change of agent is the strongest evidence it lives in the instructions rather than in one model's habits.

## The load-back test, in the wild

The fixture above tests the graduation rule. The system's actual purpose — carrying state into a session that has none — was confirmed separately in Claude Cowork, on real work rather than a fixture.

A campaign-planning project ran for a session: a targeting brief approved, an ad set drafted, the ad set then rejected, and a standing rule established and later narrowed. The session was wrapped up normally.

A **new** conversation in the same project, with no shared history, was asked what had changed. It answered:

> This is actually the start of a new session — this conversation has no prior turns of its own, so nothing's changed yet in this thread. But I found a session log from earlier today in this project folder that gives the fuller picture.

It then correctly reported that the ad set existed but was **not approved**, that the targeting brief was being revised, and which questions were still open — and it did not present the rejected ad set as finished work.

Two details worth noting:

- It distinguished *"this thread has no history"* from *"here is what happened."* Flat memory cannot make that distinction: it either recalls something or it does not, with no way to say where the knowledge came from.
- The rules it recovered came from `LESSONS.md`, the current state from `context.md`, and neither duplicated the other — the layering held without anyone checking it.

The `LESSONS.md` entry from that session is a good example of the mechanism doing something a summary would have flattened. The rule *"confirm the target job titles before drafting an ad set"* was established, then narrowed mid-session to *"only for net-new segments, not existing lookalike audiences."* The entry records the original failure, the fix, and the revised rule — so a later session inherits the rule **and** the reason it has an exception.
