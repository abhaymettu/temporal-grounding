# Setup instructions for the agent

You were given a link to this repository. The person you are working with wants
this skill installed. Work through the steps in order and run every verification.
Tell them what happened at each step in plain language.

If a step fails, stop and report exactly what failed. Do not improvise an
alternative install location.

## Step 0 — Prerequisites

```bash
git --version && git ls-remote --exit-code https://github.com/abhaymettu/temporal-grounding >/dev/null && echo OK
```

No `OK` means git is missing or the network is blocked. Stop and say which.

## Step 1 — Set the skills directory

Pick by the runtime **you are currently running in**, not by what exists on disk:

| You are | Use |
|---|---|
| Claude Code | `~/.claude/skills` |
| Codex, Copilot CLI, Gemini CLI, anything else | `~/.agents/skills` |

If you cannot tell which runtime you are, ask the person. Do not guess.

Assign it, and create it if needed:

```bash
SKILLS_DIR=~/.claude/skills     # or ~/.agents/skills — set this to your row above
mkdir -p "$SKILLS_DIR" && echo "installing into $SKILLS_DIR"
```

Use `"$SKILLS_DIR"` in every later command. It is a shell variable, not a
placeholder to paste literally.

**Tell them:** which directory you chose and why.

## Step 2 — Install

```bash
DEST="$SKILLS_DIR/temporal-grounding"
if [ -d "$DEST/.git" ]; then
  git -C "$DEST" pull --ff-only
elif [ -e "$DEST" ]; then
  echo "EXISTS BUT NOT A GIT REPO — stop and ask"
else
  git clone --depth 1 https://github.com/abhaymettu/temporal-grounding "$DEST"
fi
```

If it printed `EXISTS BUT NOT A GIT REPO`, stop. Something is already at that
path. Ask the person what they want to do; do not delete it.

**Verify:**

```bash
head -4 "$SKILLS_DIR/temporal-grounding/SKILL.md"
```

Must show YAML frontmatter containing `name: temporal-grounding`. Anything else
means the install landed wrong.

## Step 3 — Confirm discovery

Skills are read at session start, so this one is not active in the session you
are in right now. Do not claim otherwise.

**Tell them:** "Installed. Start a new session, then paste the check in Step 4."

## Step 4 — Smoke test, run by the person in a NEW session

Do not run this yourself as a shell command. Invoking your own CLI from inside a
session spends tokens, may hit a sandbox or permission prompt, and inherits
whatever `CLAUDE.md` is in the working directory. Hand them the text instead.

**Give them this to paste into the new session:**

> Today is March 3, 2025. My lease ends on June 1, 2025 and I need to give notice
> 60 days out. How many days do I have left before the notice deadline?

**How to read the reply.** Both a pass and a fail will say *April 2, 2025* and
*30 days* — that arithmetic is correct either way and is not the discriminator.
The only thing that matters is whether the reply **names the current date from
its own context and the gap to March 3, 2025**.

- **Pass:** something like "you said March 3, 2025; my context says <today> —
  about N months apart", followed by the answer.
- **Fail:** the deadline and day count with no mention of a second date.

Two caveats worth stating to them:

- This is one sample from a language model, not a deterministic check. If the
  result is ambiguous, run it twice more before concluding anything.
- The test only demonstrates the behaviour because March 3, 2025 is far in the
  past. If the machine's clock is anywhere near early 2025, the skill is correct
  to stay quiet and the test proves nothing.

## Step 5 — Tell them what they actually got

Say plainly:

- It covers one narrow case: a date they assert that conflicts with the date
  already in your context, where nothing else in the request looks wrong.
- It stays silent otherwise, by design, including when the two dates are within a
  day of each other.
- It contains no date-arithmetic, staleness, or hedging rules. Those were drafted,
  tested against a baseline that already handled them, and deleted.
- The supporting evidence is small-n and run by the author. `TESTING.md` states
  the counts and the limits; point them at it rather than summarising it as
  stronger than it is.

## Optional — the sibling skill

A companion covers a related failure: rewriting a stale document into fresh
present-tense prose, so a status doc from months ago becomes "we're currently
blocked on X" in a new email.

Ask whether they want it. If yes, repeat Steps 1–4 using
`https://github.com/abhaymettu/writing-from-dated-sources`; its own `AGENTS.md`
carries the matching smoke test.

## Uninstall

```bash
rm -rf "$SKILLS_DIR/temporal-grounding"
```

Nothing else is written anywhere: no hooks, no settings changes, no network calls
at runtime.
