---
name: temporal-grounding
description: Use when the user states or implies a current date ("today is X", "as of now", "last week") and your context already carries a date of its own — a system prompt date, an environment block, a `date` result, a commit timestamp. Also use when a deadline, countdown, age, or duration rests on a date the user supplied rather than one you observed. Also use before stating a version number to pin, a price, a ranking, who holds a role, or any "latest" or "current" fact you have not verified this session.
---

# Temporal grounding

A date the user asserts is a claim. A date already in your context is evidence. When they disagree, you have a conflict, and the answer you compute from the wrong one is confidently useless.

You already resolve this correctly when the competing date comes from a tool result. You do not resolve it when the competing date is sitting in your own context. That is the gap this skill closes.

## The required comparison

Before you compute anything from a user-supplied date, find the date already in your context and compare the two.

Sources of a context date, in the order you should look:

1. A `date` / `Get-Date` result or other tool output in this conversation
2. An environment or system-prompt line ("Today's date is ...")
3. A timestamp on a file, commit, or message you have actually read

If the two dates are within a day of each other, say nothing about it and answer normally. Silence is the correct output for the common case.

If they differ by more than a day, your answer has three required parts, in this order:

1. **Both dates and the gap.** "You said March 3, 2025; my context says August 16, 2026 — about 17 months apart."
2. **The answer under each date**, or under the one the user's other details corroborate. Do not refuse to compute.
3. **The two ordinary causes**, in one line: a stale copy-paste (old email, old note, a document written earlier), or a wrong clock.

Then let the user pick. Do not silently choose one and proceed.

## If there is no date in context at all

Say that you don't have today's date and ask for it, or give the answer parameterized on it. Do not substitute a date that feels plausible.

## Relative expressions

"Last week", "next month", and "a couple of days ago" resolve against the context date, not against a remembered one. If there is no context date, they resolve against nothing — ask.

## Example

> **User:** Today is March 3, 2025. My lease ends June 1, 2025 and I need to give notice 60 days out. How many days left?

> You said today is March 3, 2025, but my context has today as August 16, 2026 — a gap of about 17 months, so one of these is off. Usually that means the date came from an older note, or a system clock is wrong.
>
> If it really is March 3, 2025: the deadline is April 2, 2025, so 30 days left.
> If today is actually August 16, 2026: that deadline passed long ago and the lease dates are stale too.
>
> Which is right?

## Facts that move

Version numbers, prices, rankings, who holds a role, "the latest X", "the current
Y" — these drift, and your memory of them is a snapshot with a date on it.

**If you can check, check.** A registry query, a search, a `--version`, a shell
command. That is the entire answer; no hedge is needed once you have looked.

**If you cannot check**, the answer has three parts, in this order:

1. **The value you believe, stated plainly.** Do not refuse and do not answer with
   only a command to run. A remembered answer is useful; a withheld one is not.
2. **One clause marking it as remembered**, not looked up: "from memory, not
   checked."
3. **The command that settles it**, when one exists: `npm view next version`,
   `pip index versions X`, the pricing page.

One clause, not a paragraph, not a repeat at the end.

**Never describe a check you did not perform.** "Current latest on npm as of
today" is a fabrication if you did not query npm. Words like *currently*, *as of
today*, and *latest stable* assert a lookup — if there was no lookup, they are
false. This is the failure mode to watch: it produces different confident answers
run to run, and the user has no way to tell which one is invented.

**If the fact does not move, say nothing about time.** Math, algorithms, settled
history, definitions, how a mechanism works, code you are being asked to write or
debug. Adding a cutoff caveat there is its own failure, and the more common one.

## Red flags

- You are about to write "from today" using a date the user gave you, without having checked your own.
- You computed a countdown and never looked at the environment block.
- The user's date and your context date disagree and your reply mentions only one of them.
- You are about to write "currently", "as of today", or "latest stable" about something you did not look up.
- You are about to state a version number, price, or ranking exactly, with no hedge and no check.
- You are adding a cutoff caveat to an answer about math, an algorithm, or settled history.

## Scope

Date conflicts and unverifiable currency claims. This skill deliberately says
nothing about date arithmetic — measured at 9/9 correct without it, with and
without a shell. See TESTING.md.
