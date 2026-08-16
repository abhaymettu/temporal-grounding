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

## Still open

- The skill has not been tested against a *small* conflict (a day or two apart,
  e.g. a timezone boundary), where flagging is probably wrong. The one-day
  threshold in SKILL.md is a guess, not a measured value.
- Style hooks fired in every `claude -p` run despite an explicit neutralization
  instruction in `--append-system-prompt`. Output shape is contaminated; the
  temporal content is not.
- All runs are Opus 5. No cross-model check.
