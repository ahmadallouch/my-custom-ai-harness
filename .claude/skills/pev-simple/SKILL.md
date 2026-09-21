---
name: pev-simple
description: Make one small bounded change end to end without planning or issues - recon, short overview, confirm, implement, verify, PR. Works for code, docs, config, and chores alike. Reached from /pev-triage, or invoked directly when the change is obviously small. Sonnet.
disable-model-invocation: true
---

# Small change

Sonnet. Effort by feel: low for a mechanical edit, medium for a normal small change, high if there is a stack trace to chase.

No plan. No epic. No issues. No `criteria.json`. If the work turns out to need any of those, stop and say so.

This is not only for code. A README edit, a config tweak, a dependency bump, a chore - anything bounded, reversible, and free of decisions the user should make.

The request is: $ARGUMENTS

## 1. Recon

Read what the change touches. Name the files. Stop when you can describe the diff.

## 2. Overview, then confirm

Four lines. Not a plan, an orientation:

```
Change:   <what will be true after, one sentence>
Touches:  <exact file paths>
Approach: <one sentence — the mechanism, not the code>
Size:     <expected diff, e.g. "~15 lines in 2 files">
```

Then ask: "Go?" Wait. This is the only gate, so it matters.

If recon breaks one of the routing conditions - it turns out to touch a contract, a schema, a security boundary, or a decision the user should make - say so here and recommend `/pev-plan` instead. Bouncing it now is cheap; discovering it mid-diff is not.

## 3. Implement

- Do only what was asked. Zero opportunistic refactors, zero drive-by cleanups.
- Depth-1: fix only what blocks the stated change. A problem one level down gets logged, not fixed.
- No new dependencies, files, or abstractions. If you think you need one, ask.
- Choose an approach and commit to it.
- Never run git reset/checkout/clean, never --force, never --no-verify.
- Never edit `verify.sh` or `init.sh`.

Branch: `<type>/<slug>`.

## 4. Verify, proportionately

Match the check to what changed. Do not perform ceremony on a typo, and do not skip a real check on a code change.

- **Code changed**: run `verify.sh` if it exists, green, with pasted output. If it does not exist, run whatever check the repo has and say plainly that there is no gate here.
- **User-facing behaviour changed**: exercise the actual user path once and paste the result. A passing unit test is not proof the feature works.
- **Config changed**: show that the new value is actually in effect, or say explicitly that you could not confirm it and what would confirm it.
- **Docs or prose only**: no gate needed. Say that, and state what you checked instead - links resolve, commands in the README actually run, that sort of thing.
- If a check was already failing before you started, say so and do not attribute it to your change.

## 5. Ship

Commit with a descriptive message. Open a PR (operation 8 in `.claude/skills/pev-methodology/GITHUB.md`).

## Stop conditions

Stop and tell the user if:
- The change turns out to touch a schema, a contract, a security boundary, or anything a revert would not undo. → `/pev-plan`
- You need a file you did not name in the overview.
- The diff is heading past roughly double the size you quoted.
- You have failed the same check twice.
- There is a real decision to make that the user should make.

## Output

⚠️ lines first for anything assumed or stubbed. Then at most one paragraph and five bullets: what changed, what the user must verify, what is blocked. No code recap. No "next steps".
