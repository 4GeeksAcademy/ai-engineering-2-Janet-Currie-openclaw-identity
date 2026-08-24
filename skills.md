# Python-to-Java skill analysis

The skill is the 7-week Python vs Java concept map. It tells Dude how to teach Java by translating from Python J already knows.

## 1. What does this skill do? (one sentence)

It maps each week’s Python topic onto the matching Java topic so Dude can run a 10–15 minute practice session and log J’s progress in a Google Doc.

## 2. What input does the agent need?

**What you give it (in chat, usually Telegram):**

- Ask for a **daily practice session** whenever you have 10–15 minutes — e.g. `got 10 mins, quiz me` or `daily practice`.
- Optional: a week number or topic — e.g. `week 4`, `functions in Java`, or a Python snippet to rewrite. If you skip this, Dude picks from the current week in the curriculum.
- Optional extras: `put a study block on my calendar`.

You do **not** need to restate the curriculum table, Dude’s personality, J’s timezone, or the Google Doc location. Those are already in the workspace.

**What it already knows from the five configuration files:**

| File | Already known |
| --- | --- |
| `IDENTITY.md` | Agent is Dude: raven / digital familiar; sharp, mischievous, loyal, dry humor. Teaching voice stays in character. |
| `SOUL.md` | Teach by explaining *why*, not just *what*. Skip performative filler. Have opinions. Be resourceful before asking. Be careful with external writes. |
| `USER.md` | Who J is, how to address them, timezone (**CDT / GMT-05**), and that J is the learner this curriculum is for. |
| `AGENTS.md` | Operating rules: read identity/soul/user first, how to use memory, how to run a turn. |
| `TOOLS.md` / `skills.md` | Connected surfaces: Telegram replies, reminders, Google Calendar via Zapier, the progress Google Doc, and the rule to confirm external writes actually happened. |

The table itself is the skill’s **domain knowledge** (Python topic → Java topic by week). The five files supply **who**, **how to teach**, **who the student is**, **how to operate**, and **which tools can deliver the result**.

## 3. What does a good output look like?

**How a session runs:**

1. J asks for a daily practice session when there are 10–15 minutes.
2. Dude throws a code snippet (Python and/or the matching Java) and asks **what it does**.
3. After the session, Dude logs the progress **in the same Google Doc**.

**Format (Telegram, during the session):**

- One snippet, one question — not a dump of all seven weeks.
- After J answers: the *why* (typing, `static` vs instance, checked exceptions, etc.) and the matching Python/Java pair for that week.

**Format (Google Doc, after each session):**

- Date, week/topic, snippet used
- Whether J got it, what was shaky, what to hit next time
- Running log — each session appends; it does not overwrite earlier days

**Destination:**

- **Live practice:** Telegram message — snippet + “what does this do?”
- **Progress record:** Google Doc — Dude logs the session there after each one
- **Useful extra:** Google Calendar event via Zapier — a 10–15 min study block in CDT if J asks to schedule it

**How you’ll know it worked:**

- Telegram shows a snippet and a “what does this do?” question, scoped to the right week (Week 4 is `def` / methods, not `HashMap`).
- After the session, the Google Doc has a **new log entry** for that day (date, topic, result). Older entries are still there.
- The *why* is present in the follow-up (matches `SOUL.md`).
- If a calendar event was requested, it shows up on Google Calendar with the right title/time, and Telegram includes the event link — same success pattern as the “Make dinner” event.
