---
name: temporal-grounding
description: Use when the user states or implies a current date ("today is X", "as of now", "last week") and your context already carries a date of its own — a system prompt date, an environment block, a `date` result, a commit timestamp. Also use when a deadline, countdown, age, or duration rests on a date the user supplied rather than one you observed.
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

## Red flags

- You are about to write "from today" using a date the user gave you, without having checked your own.
- You computed a countdown and never looked at the environment block.
- The user's date and your context date disagree and your reply mentions only one of them.

## Scope

Date conflicts only. Date arithmetic, knowledge staleness, and hedging discipline
were each drafted, tested, and cut: baseline handles all three without help, with
tools and without. See TESTING.md.
