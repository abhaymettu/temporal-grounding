# temporal-grounding

A Claude skill for one specific failure: **you tell it a date that's wrong, and it just believes you.**

```
You:  Today is March 3, 2025. My lease ends June 1, 2025 and I need to
      give notice 60 days out. How many days do I have left?

Without the skill:
      April 2, 2025. 30 days left.
      (Correct arithmetic. The model's own context said August 16, 2026.
       It never looked.)

With the skill:
      You said today is March 3, 2025; my context says August 16, 2026 —
      about 17 months apart, usually a stale copy-paste or a wrong clock.

      Either way the notice deadline is April 2, 2025. That comes off the
      lease end date, not off today.

      From March 3: 30 days left. From August 16, 2026: it passed about
      16 months ago, and the lease dates are stale too.
```

It happens when a date gets pasted from an old email, an old note, or a document
written earlier — everything downstream is self-consistent, so nothing looks
wrong. Measured at **0/5 before, 6/6 after.**

## What it deliberately does not do

Three sections were drafted for this skill, tested, and deleted because the
baseline already handled them:

| Drafted | Result | Verdict |
|---|---|---|
| Date arithmetic rules | 9/9 computations already correct, with and without a shell | cut |
| Knowledge-staleness triage | 3/3 already volunteered "I can't confirm what's latest today" | cut |
| Anti-over-hedging rules | 0 failures in 4 attempts | cut |

Timezones and DST were also tested and left alone — the baseline correctly
counted a 1am–3am shift on fall-back night as three hours, caught that 02:30
doesn't exist on spring-forward, and knew a teammate a calendar day ahead isn't
mistaken.

The skill is ~700 words because everything without a failing test behind it was
removed.

## Design constraints

**It never blocks.** An earlier version ended by asking which date was right.
That converts your question into a question you have to answer first, so it was
rewritten: state the gap in one line, lead with whatever doesn't depend on the
date, end on the answer.

**It stays silent when the dates agree.** A gap of one day or less is more often
a timezone than an error. Verified: 3/3 runs said nothing when the user's date
was one day off, and 5/5 said nothing when it matched.

## Install

```bash
git clone https://github.com/abhaymettu/temporal-grounding \
  ~/.claude/skills/temporal-grounding
```

Claude Code discovers it automatically. For other runtimes, `~/.agents/skills/`
works as a cross-runtime alias.

## Evidence

[`TESTING.md`](TESTING.md) is the full RED/GREEN log — roughly 40 isolated runs,
including the generalization tests that showed the baseline catches *impossible*
date conflicts (a commit postdating "today") while missing *asserted* ones, and
the methodology error that produced a wrong conclusion partway through and how it
was caught.

Built with the TDD-for-skills approach from
[obra/superpowers](https://github.com/obra/superpowers): no guidance ships
without a failing test behind it.

## Related

[`writing-from-dated-sources`](https://github.com/abhaymettu/writing-from-dated-sources)
covers the adjacent failure — copying a stale document's present tense into new
prose.
