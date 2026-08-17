# Setup instructions for the agent

You were given a link to this repository. The person you are working with wants
this skill installed. Do the steps below in order, run each verification, and
tell them the result in plain language as you go. Do not skip a verification
because a step "obviously worked."

If any step fails, stop and report exactly what failed. Do not improvise an
alternative install location.

## Step 1 — Find the skills directory

| Runtime | Directory |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Codex, Copilot CLI, Gemini CLI | `~/.agents/skills/` |
| Anything else | `~/.agents/skills/` is the cross-runtime convention |

```bash
ls -d ~/.claude/skills 2>/dev/null || ls -d ~/.agents/skills 2>/dev/null || echo "NEITHER EXISTS"
```

If neither exists, create the one matching the runtime you are running in.

**Tell them:** which directory you are installing into.

## Step 2 — Install

```bash
git clone https://github.com/abhaymettu/temporal-grounding \
  <SKILLS_DIR>/temporal-grounding
```

If the directory already exists, `git -C <SKILLS_DIR>/temporal-grounding pull`
instead. Do not delete an existing directory without asking first.

**Verify:**

```bash
head -4 <SKILLS_DIR>/temporal-grounding/SKILL.md
```

The output must show YAML frontmatter with `name: temporal-grounding`. If it
does not, the clone failed or landed in the wrong place.

## Step 3 — Confirm discovery

Skills are read at session start. The person needs to start a new session before
this one is visible.

**Tell them:** "Installed. Start a new session, then I can verify it's working."

Do not claim the skill is active in the current session. It is not.

## Step 4 — Smoke test (run this in the NEW session)

This is the same scenario the skill was built from. Run it and read the output:

```bash
claude -p "Today is March 3, 2025. My lease ends on June 1, 2025 and I need to give notice 60 days out. How many days do I have left before the notice deadline?"
```

**Pass:** the reply names *both* dates — the March 3, 2025 the user asserted and
whatever the environment's current date is — and notes the gap between them,
while still giving the answer (April 2, 2025 deadline).

**Fail:** the reply says only "30 days" or similar, with no mention that the
stated date disagrees with the current date. If it fails, check that Step 2's
verification actually passed and that the session was restarted.

**Tell them** which of the two happened, quoting the relevant line.

## Step 5 — Set expectations honestly

Tell them plainly:

- This fixes one narrow thing: a date they assert that conflicts with the date
  already in your context. It stays silent otherwise, by design.
- It will not fire often. It costs nothing when it doesn't.
- It deliberately contains no date-arithmetic, staleness, or hedging rules —
  those were tested and found unnecessary. `TESTING.md` in the repo has the data.

## Optional — the sibling skill

There is a companion for a related failure: rewriting a stale document into
fresh present-tense prose (a status doc from months ago becoming "we're
currently blocked on X" in a new email).

Ask whether they want it. If yes, repeat Steps 1-4 with
`https://github.com/abhaymettu/writing-from-dated-sources`, whose own `AGENTS.md`
has its own smoke test.

## Uninstall

```bash
rm -rf <SKILLS_DIR>/temporal-grounding
```

Nothing else is written anywhere. No hooks, no settings changes, no network
calls at runtime.
