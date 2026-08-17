# Install checklist — confirm each component works once

Five components ship with this skill. This walks through each one by hand, so you can confirm the skill actually works in your environment rather than trusting that the files copied correctly.

Takes about ten minutes across two sessions. Do it once after installing.

## Before you start

- [ ] Skill directory is in place (`.claude/skills/agent-session-memory/` or `.agents/skills/agent-session-memory/`)
- [ ] `SKILL.md` is at the top of that directory
- [ ] You have run `onboarding.md` and have at least one context

---

## 1. Dated logs

**Session A.** Do a small piece of real work, then say **"wrap up"**.

- [ ] A file appeared at `.session-log/<context>/YYYY-MM-DD.md`
- [ ] It has the sections from the skill's output format
- [ ] Nothing was written inside the skill's own directory

If nothing was written: check your agent's sandbox. Codex needs memory outside `.agents/skills/`, which is why the default path is the project root.

## 2. context.md as running state

Still in session A, after the wrap-up:

- [ ] `.session-log/<context>/context.md` was **updated**, not replaced by a copy of the dated log
- [ ] Open items reflect current reality
- [ ] Resolved items were removed rather than left with strikethrough

The distinction matters: the dated log is a record of one day; `context.md` is the current state. If they contain the same text, layering is not working.

## 3. Open-items tiering

Look at `context.md`:

- [ ] `## Open items` has all three subsections — Active, Backlog, Someday / Blocked
- [ ] All three exist even when one is empty
- [ ] Something genuinely urgent is in Active, not buried in Backlog

Ask the agent: *"what should I work on in this context?"* It should answer from Active without reading everything.

## 4. Assumption graduation

The component most worth testing, because it is the one that fails silently.

During session A, say something deliberately provisional — *"I think the drop in signups is the new pricing page, but I haven't checked the analytics yet."* Then wrap up.

- [ ] It landed under `## Assumptions to validate`, **not** under Discoveries or Decisions
- [ ] It carries a tally: `seen 1×, last YYYY-MM-DD, 0 counter-examples`
- [ ] It was not stated as established fact anywhere in the dated log

If a hedged statement got recorded as a discovery, the agent is collapsing the middle state — the exact failure this skill exists to prevent.

For the full behavior test including contradiction handling, run [`graduation-fixture.md`](graduation-fixture.md).

## 5. Auto-memory (L1)

Mention a durable preference during the session — *"the client prefers a weekly summary over daily updates."*

- [ ] It went to your agent's own persistent memory, if it has one, **or** was flagged as durable rather than buried in the dated log
- [ ] It is not duplicated across the dated log and `context.md`

Agents without a persistent memory layer will note it in `context.md` instead. Acceptable — the rule is one owner per fact, not that L1 must exist.

---

## 6. Load-back (the actual proof)

**Session B — a genuinely new session.** Close the old one first.

Say: **"what did we work on last time?"**

- [ ] It read `.session-log/<context>/context.md` back
- [ ] It surfaced open items from Active
- [ ] It knew the provisional thing was provisional, not settled fact
- [ ] It did not re-derive the assumption from scratch as if new

That last box is the whole point of the system. If session B treats the assumption as either forgotten or as established fact, memory is not carrying the confidence state — which is the one thing this does that flat memory does not.

---

## If something failed

| Symptom | Likely cause |
|---|---|
| No files written at all | Sandbox blocking writes — check the memory path is outside the skills directory |
| Written to the wrong context | Context keywords in `context.md` are too vague; sharpen them |
| Assumptions recorded as discoveries | The agent skipped the layer-ownership table; re-read that section of `SKILL.md` |
| `context.md` grew past 150 lines | Pruning is not happening; it should be revised, not appended |
| Session B knew nothing | The load-mode trigger phrase did not match, or the context was misidentified |
