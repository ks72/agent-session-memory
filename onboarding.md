# Onboarding — set up your contexts

Run this once after installing. It takes a few minutes and produces the folder structure the skill writes into.

Tell your agent: **"Run the session-log onboarding."**

The agent should work through the steps below with you, then create the files.

---

## Step 1 — Name your contexts

A **context** is one ongoing stream of work that deserves its own memory. Sessions about the same context share state; sessions about different contexts stay separate.

Good contexts are things you would describe as "the X work":

- A client or employer
- A product or codebase you return to
- A side project
- A domain that spans projects (infrastructure, hiring)

Two to five is typical. Someone consulting for three clients plus their own product has four.

**Signs you have drawn the boundary wrong:**

- *Too broad* — the context's open-items list mixes work you would never do in the same week. Split it.
- *Too narrow* — you have eight contexts and half of them see one session a month. Merge the quiet ones.
- *Wrong axis* — contexts named after technologies (`react`, `python`) rather than streams of work. Memory follows projects, not tools.

Pick names that are short, lowercase, and filesystem-safe: `acme`, `internal-api`, `job-search`.

There is always a `shared` context for work that spans the others — tooling, environment setup, anything cross-cutting.

## Step 2 — Record how to recognize each context

For each context, write two or three keyword groups that identify it. The agent uses these to infer which context a session belongs to without asking every time.

Be specific enough to disambiguate. "code" identifies nothing; "checkout, payments, stripe webhook" identifies one codebase.

If two contexts share keywords, add a distinguishing term to each or accept that the agent will ask.

## Step 3 — Confirm a durable location exists, then scaffold the folders

**Nothing here creates persistent storage. It creates a folder at a path — and that path is only durable if your environment already gives you one.**

This distinction is invisible from the outside. In Claude Code and Codex, the current working directory is always a real, persistent location, so this step is automatic and this warning does not apply. **In Claude Cowork specifically, it does not hold**: without a local folder attached to the session, `.session-log/` gets created in a per-session working directory that resets between sessions. The folder gets created, files get written, the agent reports success — and none of it survives to the next session. Nothing in that sequence looks like a failure until you come back and it is gone.

Before scaffolding, confirm: is there a durable, user-attached location available right now? In Cowork, that means a local folder is attached to this project — check for one before proceeding, and if none is attached, say so and stop rather than scaffolding into working storage. Do not treat "the folder was created without an error" as proof it will still be there next session.

Once a durable location is confirmed, the agent creates this structure at the root of the project you work in — separate from wherever the skill itself is installed:

```
<project-root>/.session-log/
├── <context-1>/
│   └── context.md
├── <context-2>/
│   └── context.md
└── shared/
    └── context.md
```

Each `context.md` starts from this template:

```markdown
# <context name> — running state

**Recognize this context by:** <keywords from step 2>

## Current state
<2-4 sentences: what this work is, where it stands right now>

## Open items

### Active
- [ ] <in motion this week, or gating other work>

### Backlog
- [ ] <real and committed, not this week>

### Someday / Blocked
- [ ] <waiting on a decision or external input>

## Assumptions to validate
<!-- Provisional beliefs, each with a tally.
     Format: - <belief> — seen N×, last YYYY-MM-DD, N counter-examples
     Three INDEPENDENT corroborations promote one to a lesson.
     One counter-example resets the tally to 0. -->

## Decisions
<!-- Choices made and why. What was rejected matters as much as what was chosen. -->
```

Keep all three open-item tiers even when empty. Predictable structure means any tooling reading `### Active` never needs a special case.

## Step 4 — Seed the current state

Empty memory is not useful memory. For each context, spend two minutes writing what you already know:

- What is this work, in a sentence or two?
- What is genuinely in motion right now? (→ Active)
- What is committed but not started? (→ Backlog)
- What is parked or waiting on someone else? (→ Someday / Blocked)
- What do you *believe* about this work but have not proven? (→ Assumptions, tally 1)

That last one is the highest-value part of onboarding and the easiest to skip. Every project carries working beliefs nobody has tested — "the flaky tests are a CI resourcing problem," "that endpoint is slow because of the join." Writing them as assumptions rather than facts is exactly the discipline this skill exists to enforce. Start each at `seen 1×`.

## Step 5 — Check it works

End your next real session with **"wrap up"** and confirm:

- A dated log appeared at `.session-log/<name>/YYYY-MM-DD.md`
- `context.md` was updated rather than duplicated
- Anything provisional landed under `## Assumptions to validate`, not under a heading that implies it is established

Start the session after that with **"what did we work on last time"** and confirm it reads back.

If the agent wrote the log to the wrong context, the step-2 keywords need sharpening.

**If the second session finds nothing at all** — no error at wrap-up, just an empty read-back — the cause is very likely Step 3's warning: no durable folder was actually attached, and the files went into storage that reset between sessions. This is a different failure from the wrong-context case above and is not fixed by better keywords. In Cowork, attach a local folder to the project and redo onboarding.

---

## Notes

- **Nothing here is locked in.** Contexts can be renamed, split, or merged later — they are folders.
- **Do not port your whole history.** Seed current state only. Memory earns its value going forward.
- **`.gitignore` the contexts folder if the skill lives inside a repo you push.** Session logs record work in progress and often name clients. Keep the skill versioned; keep your memory local.
