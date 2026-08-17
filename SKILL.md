---
name: session-log
description: >
  Saves a structured memory of the current session so context carries forward to the next one.
  Use whenever the user says "session log", "wrap up", "end of session", "save my session",
  "remember this", "log what we did", "update my memory", or "what did we work on last time".
  Also trigger at the start of a new session if the user asks to recall previous context.
  When in doubt, use this skill — it is always better to save too much than to lose context.
---

# Session Log

Persistent session memory for coding agents, scoped per context.

Most agent memory is flat: one pile of facts, all equally trusted. That works until two facts contradict each other, or until a single lucky guess gets written down as a rule and every future session inherits it. This skill fixes that with two ideas:

1. **Every fact has exactly one owning layer.** Writing the same fact in three places is how memory rots — the copies drift, and nothing says which one is right.
2. **Beliefs carry a confidence state.** A guess is not a rule. It becomes a rule only after it survives repeated independent contact with reality, and one counter-example knocks it back down.

This file covers five components, complete on its own.

---

## Contexts

A **context** is one ongoing stream of work that deserves its own memory: a client, a product, a side project, a job. Sessions about the same context share state; sessions about different contexts must not bleed into each other.

Name your contexts during onboarding. Two to five is typical. Someone consulting for three clients plus their own product has four.

### Folder structure

Memory lives in `.session-log/` at the root of the project you are working in — **not** inside the skill directory:

```
<project-root>/.session-log/
├── <context-name>/
│   ├── context.md          ← running state for this context
│   ├── 2026-01-15.md       ← dated session logs
│   └── ...
├── <another-context>/
│   ├── context.md
│   └── ...
└── shared/                 ← cross-context work: infrastructure, tooling, anything spanning contexts
    ├── context.md
    └── ...
```

**Skill files and memory files stay separate, deliberately.** Two reasons:

1. **Agents sandbox their skills directory.** Codex treats skills as read-only program files and rejects writes into `.agents/skills/` under its normal `workspace-write` sandbox. Memory written there fails unless the user disables write protection, which nobody should have to do to save a session log.
2. **Updating the skill must never touch your memory.** Reinstalling or upgrading replaces the skill directory. Memory kept elsewhere survives that untouched.

If the work is not inside a project directory, use `~/.session-log/` instead and keep everything else the same.

Create missing folders and files on the fly. Never ask the user to create them.

**If you are running in an environment without a durable, user-attached filesystem** (a sandboxed agent workspace that resets between sessions, a connector-only context with no mounted folder, a cloud-only project with no local folder attached), say so before wrapping up rather than reporting a normal save. A save that reports a real path but lands in storage that will not exist next session is worse than an error — it looks identical to success until the next session comes up empty. Confirm a durable location is available; if none is, tell the user memory cannot persist here rather than proceeding as if it can.

**If a read fails, do not explain why with a theory you have not tested.** A path that returns "not found" has several possible causes — wrong location, a permissions boundary, no shared filesystem between where the user is looking and where you are running, a naming collision — and guessing one of them and stating it as the reason is a smaller version of the exact failure this skill exists to prevent: an unverified claim delivered with the same confidence as a checked one. If you can test the theory (for example, trying a variant path), test it before reporting it. If you cannot test it, say what you tried, say the cause is unknown, and say so plainly rather than naming a specific mechanism you have not confirmed.

### Identifying the active context

Before saving or loading, determine which context this session belongs to:

1. **Explicit mention** — the user says "session log for <context>" or "wrap up the <context> session".
2. **Infer from content** — match the topics discussed against the keywords you recorded for each context during onboarding. Keep those keyword sets in each `context.md` so this stays inspectable rather than guesswork.
3. **Still unclear** — ask once, listing the known context names.

**Sessions spanning multiple contexts:** write one dated log per affected context, cross-linking each with a note like "See also: `.session-log/<other>/2026-01-15.md`". Never collapse a multi-context session into a single log unless the work was purely cross-cutting with no context-specific content.

Always confirm the context before writing.

---

## Mode 1: End of Session (Save)

Triggered by: "session log", "wrap up", "save what we did", "end of session", "remember this".

### What to capture

Read back through the full conversation and extract:

1. **Session summary** — 2–4 sentences on what was accomplished. Outcomes, not process.
2. **Key decisions** — choices that should influence future work (what was chosen, what was rejected and why).
3. **Open items / next steps** — anything unfinished or explicitly deferred.
4. **Discoveries** — anything *confirmed* true about the tools, constraints, or preferences in play.
5. **Assumptions to validate** — anything believed *but not yet proven*: "I think X because Y", a probable root cause, a pattern that seems to hold. Provisional by nature. Distinct from a discovery (proven) and a decision (chosen) — this is the middle state between *noticed* and *known*. Capturing these stops the next session from either re-deriving the same guess or prematurely trusting it as fact.
6. **Files produced** — what was saved, where, and why it matters.

### Which layer owns this fact?

Write each fact to exactly ONE layer. The single biggest cause of memory rot is the same fact living in two or three places, where the copies drift apart and nothing tells you which is current. Decide the owner, write it there, and **link** from elsewhere instead of copying:

| The fact is… | Owning layer | Where |
|---|---|---|
| A durable truth about the user, a project, or how to work — worth recalling in *any* future session | **Auto-memory (L1)** | your agent's own persistent memory, if it has one |
| Current state / open items / "where we are right now" — changes often, gets pruned | **context.md (L3/L4)** | `.session-log/<name>/context.md` |
| A provisional belief not yet proven | **context.md `## Assumptions to validate`** | same file, its own section — see the graduation rule below |
| A project-specific correction — "never do X here" | **LESSONS.md (L5)** | `.session-log/<name>/LESSONS.md` |
| A deep technical explanation too costly to reconstruct from the code | **solutions doc (L6)** | the project repo's own `docs/solutions/YYYY-MM-DD-<slug>.md` |
| A time-stamped record of *this* session | **dated log** | `.session-log/<name>/YYYY-MM-DD.md` — the only append-only layer |

Rules of thumb:

- **A dated log is a record, not a home.** It may *mention* a decision, but the durable version lives in its owning layer. The log links; it does not duplicate.
- **If a fact is true across contexts, it belongs in `.session-log/shared/context.md` once** — never copied into each context.
- **State vs. truth:** if it will change next week, it is state (L3/L4). If it will still be true in six months, it is a durable truth (L1) or reference (L6). Open items and assumptions are always state.
- **When a fact moves layers** — an open item resolves into a durable rule, an assumption graduates into a lesson — delete it from the old layer. Never leave both.

### Assumptions → Lessons: the graduation rule

This is the core mechanism. Assumptions live in a `## Assumptions to validate` section of the relevant `context.md`, each carrying a tally:

```markdown
## Assumptions to validate
- Checkout test flakiness is timing-dependent, not data-dependent — seen 2×, last 2026-01-15, 0 counter-examples
```

At the start or end of a session, for each assumption you touched:

- **Corroborated** (it independently held up again, nothing contradicted it): bump the count and the date.
- **Contradicted** (a counter-example appeared): **reset the count to 0**, or delete the assumption outright if it is clearly wrong. One counter-example outweighs several confirmations.
- **Graduated** (it has cleared the threshold below): promote it — write it as a proper lesson in `LESSONS.md`, or as a durable memory or solutions doc if that fits better — then **delete it from `context.md`**.

Never let an assumption skip straight to a lesson on a single confident observation. That is exactly how a wrong "rule" gets baked in and misleads every future session.

#### What counts as independent

The threshold is **3 independent corroborations with zero counter-examples**. "Independent" is doing the real work in that sentence, so it needs a definition rather than a vibe:

> Two corroborations are **independent** when they could have disagreed. Different session, different trigger, different surrounding conditions — such that the assumption being wrong would plausibly have shown up in at least one of them.

Re-observing the same frozen situation three times inside one session is one observation looked at three times. It is not evidence of anything except that the situation did not change while you stared at it.

Two short cases, one either side of the line:

- **Graduates** — "checkout flakiness is timing-dependent" observed on three separate days, on different branches, once by accident during unrelated work. Each occasion could have contradicted it. None did.
- **Does NOT graduate** — "the staging API rate-limits per key" observed three times in one session, from one script, on one key, inside one rate-limit window. Nothing varied, so nothing could have disagreed. That tally should read `seen 1×`, not `3×`.

The diagnostic for the second case: ask what else would explain the observations. If per-IP and global limiting fit the evidence just as well as per-key, the tally has accumulated a number without narrowing anything down — the signature of a non-independent count.

Full walkthroughs of both cases, session by session: [`reference/graduation-examples.md`](reference/graduation-examples.md). Read it when applying the rule to a real assumption for the first time, or when a tally looks suspicious.

**Why this matters more than it looks.** Agents state plausible-sounding but unverified claims as fact — confidently, in the same tone they use for things they actually checked. The reset-on-contradiction rule is a structural countermeasure: a belief that has not survived independent contact with reality cannot reach the layer future sessions treat as ground truth, no matter how confident the prose around it sounded.

### Open items: tier by horizon, never by deletion

A context with real ongoing work accumulates a long `## Open items` list. That is a sign of a real backlog, not a filing problem. Do not shorten it by dropping items. Tier it by *when it is actionable*, so a reader can tell "do this week" from "true but not urgent" at a glance:

```markdown
## Open items

### Active
- [ ] Genuinely in motion this week, or gating other work

### Backlog
- [ ] Real and committed, not yet started

### Someday / Blocked
- [ ] Waiting on a decision, an external input, or explicitly parked
```

- **Active** = would come up if the user asked "what should I work on today" in this context. Keep it short — if everything is Active, nothing is.
- **Backlog** = real and intended, just not this week's problem. Most items live here; length is fine.
- **Someday / Blocked** = depends on something outside the user's control right now. Re-tier to Backlog the moment the blocker clears.
- Themed subsections can nest *under* a tier heading rather than being flattened — tier is the top-level sort, theme is secondary.
- **Tier every context uniformly, regardless of item count.** Even a short list gets all three headings, so the structure is predictable everywhere and any tooling that reads `### Active` never needs a flat-list special case.
- **A "resolved" note is not a bullet.** Anything parsing this section treats a `- ` line as an open item. Write a completed-items recap as a `>` blockquote, never a bare `- `, or it gets counted as still open.

If you wire up a start-of-session digest, have it read `### Active` first.

### Output format

Save to `.session-log/<name>/YYYY-MM-DD.md`:

```markdown
# Session Log — YYYY-MM-DD · <context name>

## What we did
[2–4 sentence summary focused on outcomes]

## Decisions made
- [Decision and brief rationale]

## Open items
- [ ] [Task or question to pick up next time]

## Discoveries
- [Confirmed preference, constraint, or context worth knowing]

## Assumptions to validate
- [Provisional belief not yet proven — omit this section if none]

## Files produced
- `path/to/file.ext` — [what it is and why it matters]
```

Then update `.session-log/<name>/context.md`, the running state file:

- Revise facts that changed.
- Add new open items; remove resolved ones.
- Maintain `## Assumptions to validate`: bump or reset tallies for assumptions touched this session, graduate any that qualify, delete graduated or disproven ones.
- Keep it under 150 lines. Prune aggressively.

Also update `.session-log/shared/context.md` if anything cross-cutting is worth noting.

If either file does not exist, create it with a minimal header.

### Extract lessons into LESSONS.md

After saving the log, silently extract lessons worth keeping. No user prompt needed.

Worth keeping:

- Errors corrected mid-session (wrong facts, wrong approach, wrong output).
- Patterns explicitly confirmed as working.
- New constraints or tool limitations discovered.

Skip anything already obvious from this file or already in `LESSONS.md`.

Append using this format:

```markdown
### SL-{NNN} — {Short title, max 8 words}
**Date:** {YYYY-MM-DD}
**Context:** {One sentence — what task was being done}
**What failed:** {Wrong approach or assumption — or "N/A — validated pattern"}
**Fix:** {What actually worked}
**Prevention rule:** {Always do / never do statement}

---
```

**The ID prefix belongs to the FILE, not to a skill or topic.** This ships with one namespace, `SL-###`, for `.session-log/<name>/LESSONS.md`. Rename it per context if you prefer (`ACME-###`), but never let two files share a prefix — duplicate IDs across files collide and break every reference that points at one. Find the next ID by scanning existing entries **in that file only**, including any `## Archived` section, since archived IDs stay taken.

Before appending to any `LESSONS.md`, read its header banner first — the file's own banner wins over anything stated here.

If no lessons are worth capturing, skip silently.

### Compact stable lessons into skill text

Lessons only change behavior if they are read at execution time, and a `LESSONS.md` with dozens of entries stops being read. After appending, check the file for compaction candidates. A lesson qualifies only when ALL of:

- Its prevention rule is a standing behavioral rule, not incident-specific forensics (incident records stay put as reference).
- It has held for roughly 30 days or several sessions with no recurrence and no counter-example.
- This SKILL.md does not already state the rule.

For each candidate: fold the prevention rule into this file as instruction text — **purely additive**. If the edit would narrow, soften, or remove an existing check, do not apply it; surface it to the user instead. Then move the compacted entry under an `## Archived — compacted` section at the bottom of its `LESSONS.md` with a one-line note (`Compacted into SKILL.md on YYYY-MM-DD`), keeping its ID.

At most 1–2 compactions per wrap-up. Steady maintenance, not a rewrite pass.

### Confirm to the user

> "Saved session log to `.session-log/<name>/YYYY-MM-DD.md` and updated context. Open items: [brief list]. Lessons extracted: [N] or none."

---

## Mode 2: Start of Session (Load)

Triggered by: "what did we work on last time", "load my context", "pick up where we left off", "what's the status of <context>", "remind me where we were".

1. Identify the context (same logic as above).
2. Read `.session-log/<name>/context.md` and summarize the most relevant points.
3. Check the most recent dated log and surface any open items.
4. Say what was loaded, then ask what to work on.

If no memory files exist yet, say so and offer to set them up at the end of the session.

**Known limitation (v1):** loading is a targeted read of the current context's files, not a search. There is no cross-context search and no automatic recall of arbitrary past dated logs — if a detail lives in a log from three months ago and you cannot name the context, this will not surface it. Recent state carries forward reliably; deep history does not. Built-in memory in Claude and ChatGPT does provide broad recall, and this does not replace it.

---

## Notes

- Dated logs are a historical record. Never edit a past log; write a new one.
- Multiple sessions in one day: use a `-part2`, `-part3` suffix. **Re-list the target folder immediately before writing** — never trust a listing from earlier in the session, since a parallel session in another window may have created today's file in the meantime. On collision, read the existing file first, then pick the next suffix and fix any cross-links.
- Keep `context.md` evergreen. Prune resolved items rather than appending forever.
- If the user is in a hurry, ask: "Quick wrap-up or full log?" A brief log beats none.
- Target log length: 30–150 lines. Under 30 means something was missed; over 150 means it needs cutting.
