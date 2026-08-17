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
SKILLS_DIR=~/.claude/skills   # or ~/.agents/skills — pick from the table above
mkdir -p "$SKILLS_DIR" && echo "installing into $SKILLS_DIR"
```

**Every command block below re-declares `SKILLS_DIR` on its own first line.** That
is deliberate: shell state does not survive between tool calls, and an unset
`SKILLS_DIR` would turn the install and uninstall commands into operations on `/`.
Edit the value in each block to match what you chose here. Do not delete the
line, and do not assume it carries over.

**Tell them:** which directory you chose and why.

## Step 2 — Install

```bash
SKILLS_DIR=~/.claude/skills   # same value as Step 1
set -u
: "${SKILLS_DIR:?set SKILLS_DIR on the line above}"
DEST="$SKILLS_DIR/temporal-grounding"
if [ -d "$DEST/.git" ]; then
  if [ -n "$(git -C "$DEST" status --porcelain)" ]; then
    echo "LOCAL EDITS PRESENT — stop and ask before updating"
  else
    git -C "$DEST" fetch --depth 1 origin main && git -C "$DEST" reset --hard origin/main
  fi
elif [ -e "$DEST" ]; then
  echo "EXISTS BUT NOT A GIT REPO — stop and ask"
else
  git clone --depth 1 https://github.com/abhaymettu/temporal-grounding "$DEST"
fi
```

(`fetch` + `reset` rather than `pull`, because `--depth 1` clones are shallow and
`pull --ff-only` is unreliable against them.)

If it printed `LOCAL EDITS PRESENT`, someone has customised this copy — the
update would overwrite their changes. Stop and ask; do not force it.

If it printed `EXISTS BUT NOT A GIT REPO`, stop. Something is already at that
path. Ask the person what they want to do; do not delete it.

**Verify:**

```bash
SKILLS_DIR=~/.claude/skills   # same value as Step 1
head -4 "$SKILLS_DIR/temporal-grounding/SKILL.md"
```

Must show YAML frontmatter containing `name: temporal-grounding`. Anything else
means the install landed wrong.

## Step 3 — Confirm discovery

In Claude Code, skills are read at session start, so this one is not active in
the session you are in right now — do not claim otherwise.

**Only Claude Code is verified.** `~/.agents/skills` is a convention other
runtimes may read, but nothing here has been tested on Codex, Copilot CLI, or
Gemini CLI, and how they load skills is their business, not this repo's. If you
are on one of those, tell the person plainly that discovery is unverified and
point them at their runtime's own docs before running any check.

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

This runs in a session you cannot see. Ask them to paste the reply back, judge it
against the criteria above, and tell them the verdict. Do not report a result you
have not been shown.

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

Ask whether they want it. If yes, follow that repo's own AGENTS.md — do not reuse the commands here, they hardcode this repo's URL and directory name:
`https://github.com/abhaymettu/writing-from-dated-sources`.

## Uninstall

```bash
SKILLS_DIR=~/.claude/skills   # same value as Step 1
rm -rf "${SKILLS_DIR:?set SKILLS_DIR first}/temporal-grounding"
```

Nothing else is written anywhere: no hooks, no settings changes, no network calls
at runtime.
