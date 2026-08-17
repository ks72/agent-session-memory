# agent-session-memory — for Claude Code, Codex, Claude Cowork, and ChatGPT Work

A memory skill for coding agents. It keeps every fact in one place, and it tells the difference between "I'm pretty sure" and "I checked this." A guess only becomes a rule after it holds up more than once — and one wrong result knocks it back down.

Works in **Claude Code, Codex, Claude Cowork, and ChatGPT Work**. Written to the [open agent skills standard](https://agentskills.io), so it isn't locked to one product.

---

## Who is it for

Anyone already working with Claude Code, Codex, Cowork, or ChatGPT Work across more than one ongoing project — consultants, freelancers, and marketers juggling several clients or campaigns, not just engineers.

If you've ever had your agent forget something you told it last week, or state a guess as if it were confirmed, this is for you.

You don't need to code to use it. You do need to be comfortable installing a skill the normal way for your platform (see [Install](#install) below).

## Why

Most agent memory is one flat pile of facts, all trusted equally. That breaks in two ways, and both get worse the longer you use it:

- **The same fact ends up in three places.** A decision lands in a project file, a chat summary, and the agent's own memory. One copy changes. Now there are three versions of the truth and no rule for which one is current — so the agent picks whichever one it sees first.
- **A guess hardens into a rule.** The agent notices something once, writes it down as confidently as something it actually checked, and every future session treats it as fact. Nobody double-checks it, because it already reads as settled — so the mistake can steer your work for weeks before anyone catches it.

Agents describe a hunch and a verified fact in exactly the same tone. There's no difference between "I checked this" and "this seems right." If memory has no place to record that difference, it's gone the moment the note gets written.

*(If you have more time: see [how this compares to Claude and ChatGPT's built-in memory](#compared-to-built-in-memory).)*

## What it produces

Every session writes to up to three plain text files, each with one job:

```
.session-log/<your-context>/
├── 2026-01-15.md      ← today's log
├── context.md          ← current state
└── LESSONS.md          ← rules learned over time
```

- **`2026-01-15.md` — the log for that day.** What happened, what was decided, what's still open, and anything believed but not yet proven. Written once and never edited again — a diary entry, not a place to update later.
- **`context.md` — where things stand right now.** Updated every session, not appended to. Holds the current open items (sorted by how urgent they are), the running list of unproven beliefs and their tallies, and any durable facts about the work. This is the file a new session reads first.
- **`LESSONS.md` — rules that have earned their place.** A belief only lands here after it's been checked three separate times with nothing contradicting it. Each entry says what was tried, what went wrong, and the rule that came out of it.

These live at the root of the project you're working in — **not** inside the skill folder:

```
<skills-root>/session-log/        ← the skill itself (replaced when you update it)
├── SKILL.md
├── reference/
│   └── graduation-examples.md
└── onboarding.md

<project-root>/.session-log/      ← your memory (written to every session)
├── <your-context>/
│   ├── context.md
│   └── 2026-01-15.md
└── shared/
    └── context.md
```

`<skills-root>` is `.claude/skills/` or `.agents/skills/`, depending on your platform.

**Why memory isn't stored inside the skill folder:** Codex won't let a skill write files into its own skill folder — that's a safety rule, confirmed by testing, not a guess. Keeping memory at the project root sidesteps that entirely, and it means updating the skill later never touches anything it has already saved.

If your project is a repo you push to GitHub, add `.session-log/` to `.gitignore` — session logs describe work in progress and often name real clients.

See [`examples/worked-session.md`](examples/worked-session.md) for a real before-and-after: a raw session summary, and what these three files look like once it's been sorted into them.

---

## Install

The skill is a folder containing `SKILL.md`. Where you put it depends on your platform.

### Claude Code

```bash
git clone https://github.com/ks72/agent-session-memory.git
mkdir -p .claude/skills
cp -r agent-session-memory .claude/skills/session-log
```

Use `~/.claude/skills/session-log` instead if you want it available in every project, not just this one.

### Codex

Codex looks in a few places, depending on scope ([source](https://learn.chatgpt.com/docs/build-skills)):

- This project only: `$CWD/.agents/skills` or `$REPO_ROOT/.agents/skills`
- Every project on your machine: `$HOME/.agents/skills`

```bash
mkdir -p .agents/skills
cp -r agent-session-memory .agents/skills/session-log
```

### Claude Cowork

1. In Cowork, go to **Customize → Skills** and upload the skill folder as a `.zip`.
2. **Attach a real local folder to the project before you use the skill.** Without one, Cowork will report "saved" but the file quietly disappears between sessions. Details and the fix: [Cowork setup](#cowork-attach-a-folder-first).

### ChatGPT Work

1. In the ChatGPT desktop app, create or open a **local project** — not a regular (cloud) project.
2. Attach the folder you want this skill to write to.
3. Install the skill and run onboarding as normal. Details and the fix: [ChatGPT Work setup](#chatgpt-work-use-a-local-project).

### After installing, on any platform

Run [`onboarding.md`](onboarding.md) once to name your contexts and set up their folders.

## Use it

Say **"wrap up"** or **"session log"** at the end of a session. The agent writes a dated log, updates the running notes for that context, and adjusts the tallies.

Say **"what did we work on last time"** at the start of a new one. It reads the notes back.

## Try it yourself

Two files ship with this repo so you can check it's working, not just take our word for it:

- [`examples/install-checklist.md`](examples/install-checklist.md) — walk through all five parts by hand
- [`examples/graduation-fixture.md`](examples/graduation-fixture.md) — confirm a contradiction actually resets a tally instead of being ignored

## More info? issues? questions?
Contact me, i'll try to do my best to help.
Ludo@ritmoo.io

---

## If you have more time

### What it does

Five parts, all just plain text files:

- **Contexts** — one memory space per stream of work (a client, a project, a job). They stay separate.
- **One owner per fact** — every fact lives in exactly one place. Other files link to it instead of copying it.
- **Assumption grading** — a guess gets its own section with a tally. It takes three independent confirmations to become a rule. One contradiction resets it to zero.
- **Open items by urgency** — work is sorted into Active, Backlog, and Someday, so nothing important gets buried and nothing old gets deleted.
- **Dated logs** — a running diary per session. It mentions decisions; it doesn't own them.

A tally in `context.md` looks like this:

```markdown
## Assumptions to validate
- Signup drop-off is caused by the new pricing page, not the checkout flow — seen 2×, last 2026-01-15, 0 counter-examples
```

That line is a belief the agent isn't allowed to treat as fact yet. It has a tally, a date, and a count of times it's been proven wrong. It only becomes a rule once it clears the bar — and drops back to zero the moment something contradicts it.

#### What "independent" actually means

The rule is three independent confirmations. "Independent" is the word doing all the work, so here's the plain definition:

> Two confirmations are independent when they *could* have disagreed. Different session, different trigger, different circumstances — such that if the belief were wrong, at least one of them would likely have shown it.

Two examples, one on each side:

- **Becomes a rule:** "the signup drop was the new pricing page, not checkout" — confirmed on three separate occasions, by three different kinds of evidence (a holdout page, a revert, a re-check on a different day). Each time, something could have proven it wrong. Nothing did.
- **Stays a guess:** "the MQL-to-SQL drop is a lead-quality issue" — seen three times in one sitting, from one CRM report, viewed three different ways. Nothing changed between the three checks, so nothing could have disagreed. That should be logged as `seen 1×`, not `seen 3×`.

The quick test: ask what else could explain what you saw. If two different explanations both fit everything you observed, the tally has grown without actually telling you anything — a sign the confirmations weren't independent.

Full worked examples: [`reference/graduation-examples.md`](reference/graduation-examples.md).

### Benefits — and what it doesn't do

**What you get that flat memory doesn't give you:**

- A place for "I think this" that's distinct from both "this is true" and "not recorded at all."
- Being wrong actually costs something — one contradiction wipes out an accumulated tally.
- You can see exactly why a rule became a rule, and undo it if you disagree.
- No duplicate copies drifting apart — every fact has one home.
- Client A's notes never leak into Client B's work.

**What it doesn't do:**

- **No search across past sessions.** It reads the current project's own files — it doesn't dig through history or hunt down an old note from three months ago. Built-in memory in Claude and ChatGPT does this well already; this skill isn't trying to replace that.
- **Grading a tally still needs judgment.** The independence rule is spelled out with examples, but the agent still has to apply it. A careless agent can bump a tally that shouldn't move.

### Compared to built-in memory

Claude and ChatGPT both come with their own memory. They're good at different things, and this skill isn't a replacement for either — it fills a specific gap.

| | Claude memory | ChatGPT memory | This skill |
|---|---|---|---|
| **How it stores things** | Separate entries by category, updated as you chat ([source](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context)) | One evolving summary, plus an older "saved memories" mode ([source](https://help.openai.com/en/articles/8590148)) | Plain text files you can read, compare, and back up yourself |
| **Where it applies** | Per project, plus one running summary refreshed daily | Across your whole account | Per context, defined by you |
| **Tracks confidence** | Not documented | Not documented | Yes — a tally and a contradiction count |
| **On contradiction** | Not documented | Updates the summary automatically, but doesn't show which belief won or why | Tally resets to zero, and you can see it happen |
| **Searches past sessions** | Yes | Yes | **No** — see above |
| **Works in Claude Code** | **No** — Claude's memory only runs in the Claude.ai app, not Claude Code ([source](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context)) | n/a | Yes — that's the whole point |

The claim here is narrow, on purpose:

- **This isn't a claim to be "better" than built-in memory.** It solves one specific problem neither product documents a fix for — there's no in-between state for a belief, and no clear rule for what happens to confidence once something's proven wrong. ChatGPT's own docs admit the old system let entries contradict each other (their example: "I'm training for a marathon" next to "I sprained my ankle"), and say the newer system fixes that by updating the summary automatically. But that update happens quietly, with no record of which belief won or how sure it was — which is the part this skill writes down.
- **For Claude Code users, there's no real comparison to make** — Claude's built-in memory doesn't run there at all. The nearest thing is `CLAUDE.md` files, which hold fixed instructions, not accumulated state that changes over time. This fills an actual gap rather than competing with something already there.

#### And against auto-memory that runs per project

Some products now build memory automatically per project — Claude Projects, and Cowork, where project memory is described as *"what Claude learns from the conversations you have inside them… **you don't write it**"* ([source](https://academy.claude.com/courses/introduction-to-claude-cowork/giving-cowork-context)).

That's the whole difference in one phrase. Auto-memory and this skill aren't competing — they fall short in different places.

| | Auto-memory | This skill |
|---|---|---|
| Who writes it | The agent, automatically | You and the agent, on purpose |
| Effort | None | A minute or two per session |
| Can you inspect it | As a summary | As a plain file you can compare version to version |
| Tracks confidence | Not documented | Yes |
| On contradiction | Updated quietly | Tally resets to zero, visibly |
| Stays with you if you switch tools | No | Yes — it's just files |

Auto-memory is genuinely good at the thing nobody wants to do by hand — noticing and keeping context with zero effort. Keep using it. What it can't do is hold *"I believe this, but haven't proven it"* — a summary records what's believed, not how well-tested that belief is.

Use both together: let auto-memory handle the everyday context, and use this for the beliefs you want checked before they become rules.

### Platform status

| Platform | Status |
|---|---|
| Claude Code | **Confirmed working.** Followed the instructions correctly on its own; memory saved and read back with no issues. |
| Codex | **Confirmed working** (v0.144.6). Found the skill on its own, ran a full session, saved memory correctly. |
| Claude Cowork | **Confirmed working, with one requirement.** Needs a local folder attached to the session — see below. |
| ChatGPT Work | **Confirmed working, local project only.** Needs a local project, not a regular one — see below. |
| Claude.ai chat | **Not expected to work.** Chat sessions are sandboxed per conversation, so nothing saved there carries over to the next one. |

Nothing on this list is marked as working without having actually been tested there. This table reflects real results, not guesses from reading the docs.

#### Cowork: attach a folder first

**This only works in Cowork if you attach a real local folder to the session first.**

Without one, the skill will still say "saved" — it even names a real-looking file path — but that file goes into Cowork's temporary session storage instead of somewhere permanent. That storage clears out between sessions, so the file is gone by your next conversation.

This is worse than a normal error, because nothing looks wrong when it happens. It was confirmed by testing: a session with no folder attached, using a connector (like Dropbox) as its only source of context, reported a successful save that had vanished by the next session.

**What to do:** before using this skill in Cowork, attach a local folder to the project — the same one every time. A connector can bring information *in*, but the skill can't write memory *to* a connector. Skip this step and the skill will look like it's working for exactly one session, then quietly stop.

The skill can't create this folder for you — onboarding sets up `.session-log/<context>/` at a location, and whether that location survives between sessions depends entirely on your platform. Onboarding now checks for a real folder before setting anything up, so in Cowork, attach the folder first, then run onboarding.

#### ChatGPT Work: use a local project

ChatGPT has two different things both called a "project," and only one of them can save files to your computer:

- **Regular (cloud) project:** no folder access at all. OpenAI's own docs say it plainly: *"A ChatGPT project doesn't provide direct access to a folder on your computer, so upload or connect the sources you want ChatGPT to use."*
- **Local project** (desktop app only): connects to a real folder on your computer, and ChatGPT can read and write files there directly.

**Use a local project.** In a regular one, you'll hit the same problem as an unattached Cowork session — the skill reports a save that has nowhere real to land, and your next session finds nothing. Tested and confirmed working in a local project with a folder attached.

To set one up: in the ChatGPT desktop app, create or open a local project (not a regular one), attach the folder you want the skill to use, then install and run onboarding normally.

### What's next (not built yet)

Ideas worth noting, not a roadmap or a promise:

- Checking a session's decisions against other project docs that might now be out of date.
- A tagged log of repeat mistakes, separate from written-out lessons, so a recurring problem is easy to spot.
- Scoring each session against a quality checklist and tracking that over time.

A plugin version for each platform — rather than copying a folder by hand — is on the wishlist too, not built yet.

---

## License

MIT — see [LICENSE](LICENSE).
