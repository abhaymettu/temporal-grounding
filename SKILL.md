---
name: temporal-grounding
description: Use when the user states or implies a current date ("today is X", "as of now", "last week") and your context already carries a date of its own — a system prompt date, an environment block, a `date` result, a commit timestamp. Also use when a deadline, countdown, age, or duration rests on a date the user supplied rather than one you observed.
---

# Temporal grounding

A date the user asserts is a claim. A date already in your context is evidence. When they disagree, you have a conflict, and the answer you compute from the wrong one is confidently useless.

You already resolve this when the conflict is internally impossible — a commit postdating "today", a duration that cannot fit. You miss it when the wrong date is merely asserted and every other number is self-consistent. That gap is what this skill closes.

## The required comparison

Before you compute anything from a user-supplied date, find the date already in your context and compare the two.

Sources of a context date, in the order you should look:

1. A `date` / `Get-Date` result or other tool output in this conversation
2. An environment or system-prompt line ("Today's date is ...")
3. A timestamp on a file, commit, or message you have actually read

If the two dates are within a day of each other, say nothing about it and answer normally. Silence is the correct output for the common case. A one-day gap is more often a timezone than an error.

If they differ by more than a day, **answer anyway**. Do not stop and ask which date is right — that turns their question into your question and they came here to get an answer. Three parts, in this order:

1. **One line naming both dates and the gap**, with the likely cause: "You said March 3, 2025; my context says August 16, 2026 — about 17 months apart, usually a stale copy-paste or a wrong clock."
2. **The part of the answer that does not depend on which date is right.** A renewal date, an expiry, a deadline computed from other fixed dates reads the same under either. This is often most of the answer.
3. **The part that does depend on it** — under both readings, or under the one the user's other details corroborate, saying which you used.

End on the answer, not on a question. If the date needs correcting, they will correct it.

## If there is no date in context at all

Say that you don't have today's date and ask for it, or give the answer parameterized on it. Do not substitute a date that feels plausible.

## Relative expressions

"Last week", "next month", and "a couple of days ago" resolve against the context date, not against a remembered one. If there is no context date, they resolve against nothing — ask.

## Example

> **User:** Today is March 3, 2025. My lease ends June 1, 2025 and I need to give notice 60 days out. How many days left?

> You said today is March 3, 2025; my context says August 16, 2026 — about 17 months apart, usually a stale copy-paste or a wrong clock.
>
> Either way the notice deadline is **April 2, 2025** — that comes off the lease end date, not off today.
>
> From March 3, that's 30 days left. From August 16, 2026, it passed about 16 months ago and the lease dates are stale too.

## Red flags

- You are about to write "from today" using a date the user gave you, without having checked your own.
- You computed a countdown and never looked at the environment block.
- The user's date and your context date disagree and your reply mentions only one of them.
- You are ending a reply by asking which date is right instead of answering under both.

## Scope

Date conflicts only. Date arithmetic, knowledge staleness, timezone and DST
handling, and hedging discipline were each drafted or tested and cut: baseline
handles them, with tools and without. Writing fresh prose from a dated source is
a separate measured failure, split into the `writing-from-dated-sources` skill.
See TESTING.md.
