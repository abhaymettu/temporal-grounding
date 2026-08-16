# Baseline evidence

RED-phase results. 14 subagent runs, no skill loaded, Claude Code + Opus 5 with
shell and web search available. Environment date 2026-08-16.

## What failed

**Context-date conflict — 5/5 runs failed.**

Prompt: "Today is March 3, 2025. My lease ends on June 1, 2025 and I need to give
notice 60 days out. How many days do I have left before the notice deadline?"

Every run computed April 2, 2025 / 30 days — arithmetically correct. Not one
noticed that the user's asserted date sits 531 days before the date in its own
environment block. All five wrote "from today" while meaning the user's date.
Four of the five were run with terseness hooks neutralized, so this is not a
style artifact.

**The contrast that defines the rule.** A sixth scenario put the conflicting date
in a *tool result* instead of the context: "Run `date` to confirm today, then
tell me how many days since 2026-08-01. Today's October 2, 2026 by the way."
That run passed cleanly — it ran `date`, got 2026-08-16, said the clock disagreed
by six and a half weeks, gave both counts (15 and 62), named both causes, and
warned that a skewed clock would also corrupt the commit timestamps.

So the capability is present; it just isn't applied to the model's own context
date. That is why SKILL.md is scoped to the comparison step rather than to
date reasoning in general.

## What did not fail

Three sections of the original draft had no failing test behind them and were cut.

**Date arithmetic — 0/8 wrong.** Eight independent date computations across the
scenarios, all exactly right against `python3 datetime` ground truth. The
hardest one (visa expiry 2027-01-14, minus 90 days, minus 6 weeks, plus weekday)
returned Friday 2026-09-04, which is correct. Manual counting rules would be
teaching a skill already held.

**Knowledge staleness — 0/2 failed.** Asked for the latest Claude model and
price, the baseline looked it up and answered specifically. Asked what Next.js
version to pin, it declined to name one and returned `npm view next version`
instead — arguably over-cautious, but the opposite of the failure the draft
guarded against.

**Over-hedging — 0/4 failed.** Quicksort complexity and an age-from-birthdate
function, run both with and without terseness hooks. No cutoff caveats, no
"verify this", no date references. The long unhooked quicksort answer ran ~700
words with zero temporal hedging.

## Known limits of this evidence

- One harness. These runs had a shell and web search. In a bare chat with no
  tools, the arithmetic and staleness sections may well earn their place — retest
  there before deleting the idea permanently.
- n=5 on the failing scenario, n=1–4 on the passing ones. The passes are weaker
  evidence than the failure.
- Subagents inherit the parent session's skills. Round 1 showed clear ponytail
  style bleed; round 2 added an explicit neutralization instruction, and the
  over-hedging result held in both.

## GREEN phase

8 runs with SKILL.md on disk and discoverable. No instruction to use it — every
run found and invoked it on its own, so discovery via the description works.

**Conflict scenario — 4/4 pass** (was 0/5). Every run produced all three required
parts in order: both dates and the gap, the answer under each, the two ordinary
causes. Wording converged tightly across reps, which is the signal that the form
is binding rather than being reinterpreted each time.

**No-conflict control — 3/3 pass.** "Today is August 16, 2026" against an
environment date of 2026-08-16. All three answered normally — 17 days, deadline
September 2, 2026 — and none announced that it had compared dates. This was the
real risk: a required-comparison rule is exactly the shape that leaks into the
common case. It didn't.

**No user-supplied date — 1/1 pass.** Same lease question with no date given. Used
the environment date silently and stated it as a fact ("Today is August 16, 2026")
rather than narrating a lookup.

Arithmetic stayed correct throughout, including the secondary counts the runs
volunteered (Sept 1 → 16 days, Aug 31 → 15 days).

## Toolless phase

Run via `claude -p` in tmux to answer the open question above: do the cut
sections earn their place when there is no shell and no search?

**A false result first, recorded because the error is instructive.** The first
two attempts used `claude -p --tools ""` and `--tools "Skill"`. That flag only
restricts *built-in* tools. Every MCP server stayed live, including
`mcp__fetch__fetch`. Runs that looked like they were answering a version
question from memory were in fact querying the npm registry. On that basis a
staleness section was written and added to SKILL.md — one run was even accused
of fabricating "current latest on npm as of today" when it had genuinely
fetched it and was right.

Isolation requires both flags:

    claude -p "$PROMPT" --tools "Skill" --strict-mcp-config --mcp-config '{"mcpServers":{}}'

Verified by asking a run to list its tools: with both flags it reports `Skill`
and nothing else. With only `--tools`, it reports fetch, filesystem, Playwright,
Gmail, Calendar, Drive, LinkedIn, Context7, and Tsenta.

**Staleness, properly isolated — 3/3 pass, skill removed from disk.** Asked what
Next.js version to pin with no way to check, all three runs volunteered the
limit unprompted: "my knowledge ends May 2026", "I can't confirm what's latest
today", and a placeholder `16.x.y` with `npm view next version` to resolve it.
None asserted a specific patch version as current. There is no failure here to
write a skill against, so the staleness section was removed again.

**Over-hedging leak — pass.** Quicksort complexity, skill present vs absent:
0 temporal hedges with the skill, 1 without. The skill does not induce caveats
on non-decaying facts.

**Date conflict, properly isolated — 2/2 full compliance**, all three required
parts. The conflict rule is the one thing here that works without tools.

**One-day threshold — 3/3 pass.** User asserts August 15, 2026 against a context
date of August 16, 2026. The skill says stay silent under a one-day gap, and all
three runs did: no comparison mentioned, user's date used, 18 days to a September
2 deadline, arithmetic correct. The threshold was a guess when written; it now
has a measurement behind it at exactly one day. Two days and wider is still
untested.

## Generalization phase

Three conflict shapes the rule was not derived from, isolated, skill moved off
disk for the RED arm, n=2 per arm.

**W1 — user date four months ahead of context.** "Today is December 20, 2026,"
warranty runs 18 months from June 1, 2026. RED **2/2 caught it** unprompted, one
rep giving both countdowns (346 vs 472 days) and asking which date is right.
GREEN 2/2, slightly more structured. No gain.

**W2 — a third date source contradicting both.** "Today is May 2, 2026," with a
pasted commit dated Jul 6, 2026, against a 30-day archive policy. RED **2/2
caught it**, both spotting that a branch cannot hold a commit from the future.
GREEN 2/2, one rep enumerating both conflicts separately. No gain.

**W3 — user date asserted in passing, with corroborating detail.** "It is October
5, 2026 (I am still recovering from the concert on the 3rd)," membership renews
90 days after August 1. RED **0/2** — both answered "25 days from today" off the
user's date with no flag at all. GREEN **2/2**, and both noted the renewal date
itself (Oct 30) is invariant while only the countdown moves. Clear gain.

### What the three results mean together

The baseline is not uniformly blind to date conflicts. It reliably catches a
conflict that is *internally impossible* — a commit postdating "today", a
duration that cannot fit. It misses a conflict that is merely *asserted*, where
every number is self-consistent and only the anchor is wrong.

Put plainly: **baseline catches contradictions and misses substitutions.** Both
scenarios with a measured gain (the lease case and W3) are substitutions. Both
scenarios with no gain (W1, W2) contain an internal impossibility the model can
find without ever consulting its context date.

Direction is not the variable. W1 and W3 both put the user's date in the future
relative to context, and only W3 slipped through. The difference is that W3's
date arrives inside a casual aside with a corroborating detail attached. The
corroboration appears to suppress the check rather than invite it, which is the
opposite of useful.

## Application sweep

The scenarios so far were all one shape: ephemeral chat Q&A where the user hands
over a date and asks for a countdown. Four agents each proposed six realistic
failures in an area that shape does not cover — durable artifacts, dates living
inside pasted content, timezones and DST, and data recency plus stale agent
sessions. Sixteen were run RED, isolated, skill off disk.

**14 of 16 passed.** Baseline handled far more than expected:

- DST: shift across fall-back correctly counted as 3 hours, not 2. The
  nonexistent 02:30 on spring-forward was caught, Phoenix's lack of DST included.
- A teammate a calendar day ahead was called correct rather than mistaken.
- A zoneless log line was resolved against the actual wall clock and identified
  as stamped in the *future*.
- A stale session plan file was caught outright: "the plan file is stale: freeze
  is 2 days out, not 7."
- `now() - interval '2 years'` rather than a date literal; an injected clock in a
  pytest rather than a wall-clock assertion; a changelog dated from the context
  date correctly; fiscal Q3 with 76 days remaining, all correct.
- An overdue TODO, a missed cert rotation, and a 16-month-old onboarding doc were
  each flagged unprompted.

**The 2 failures share a shape neither the axes nor the scenario authors
predicted.** Both are generation, not analysis:

- Asked to write a client email from a `STATUS.md` dated 2025-11-20, the output
  was present tense throughout — "We're in integration testing", "One item is
  blocking us" — with no mention anywhere that the source was nine months old.
- Asked whether MAU is growing "right now" from a table ending 2025-11, it
  correctly refused the trailing-12 average, then answered "growing, but
  decelerating" in the present tense without noting the data ends nine months back.

**The rule this yields: the model catches a stale document when asked to judge
it, and transcribes it when asked to rewrite it.** Every evaluation task in the
sweep passed. Both generation tasks failed. A source's "currently" is a claim
about the source's date; copying it forward silently reasserts it as a claim
about today, inside an artifact that then travels to someone else.

## Two changes, both tested

**Non-blocking conflicts.** The earlier text ended with "let the user pick, do
not silently choose one and proceed." That blocks — it converts the user's
question into a question they must answer first. Rewritten to state the gap in
one line, lead with whatever does not depend on the anchor, give the dependent
part under both readings, and end on the answer. GREEN 2/2 on the lease and gym
conflicts: both ended on the answer, neither asked the user to choose, and both
put the invariant first (the notice deadline and the renewal date come off other
fixed dates, not off today).

**Writing from a dated source.** New section. GREEN: the status email now carries
its age in the subject line and body — where it travels to the recipient — not
just in the reply to the requester. The MAU answer now says the data is ~9 months
stale and attributes the trend to November 2025.

**Leak control and the fix it forced.** A control ran the same status doc dated
2026-08-14, two days old. First GREEN attempt leaked: it announced "Status doc is
dated Aug 14, today is Aug 16, so it's 2 days old" before the email — exactly the
added friction the skill is supposed to avoid. The generation section had no
silence threshold, unlike the conflict rule. Added one, and re-ran: 2/2 now write
the email straight with no age preamble, while the nine-month case still flags.

## Split

Once the sweep showed the generation failure was a different failure with a
different trigger, the two were split. `temporal-grounding` keeps the date
conflict rule (707 words); `writing-from-dated-sources` takes the dated-source
rule (643 words). Combined they had reached 1051 words in one file, well over the
500-word guideline, and the two halves fire on disjoint requests — a countdown
question never needs the generation rule, and a "write me an email from this doc"
request never needs the conflict rule.

Re-tested after the split, 5 scenarios: stale doc flags, stale data flags, fresh
doc writes clean with no age preamble, no-conflict countdown stays plain,
conflict flags and leads with the invariant. No regression from the split.

## Still open

- Cross-harness: only Claude Code. Untested in claude.ai, the API, or any other
  runtime that surfaces a date differently.
- Style hooks fired in every `claude -p` run despite an explicit neutralization
  instruction in `--append-system-prompt`. Output shape is contaminated; the
  temporal content is not.
- All runs are Opus 5. No cross-model check.
