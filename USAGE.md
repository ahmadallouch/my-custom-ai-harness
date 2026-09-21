# PEV - Plan / Execute / Verify

Six Claude Code skills plus an instruction-file block. Opus plans and closes every decision, GitHub issues carry the plan, Sonnet executes one issue per fresh chat, a script decides pass/fail and a read-only verifier decides whether the right thing was built.

## Contents

```
.claude/skills/
  pev-methodology/SKILL.md      the loop, artifacts, ownership (model-invocable)
  pev-methodology/REFERENCE.md  tiers, blast radius, model matrix, trust boundary, anti-drift
  pev-methodology/GITHUB.md     the eight GitHub operations, bound to MCP or gh CLI
  pev-triage/SKILL.md           /pev-triage <request>      Opus, medium — START HERE
  pev-advise/SKILL.md           /pev-advise <request>      Opus, high — investigate, recommend, change nothing
  pev-simple/SKILL.md           /pev-simple <request>      Sonnet — one small bounded change
  pev-bootstrap/SKILL.md        /pev-bootstrap             once per repo
  pev-plan/SKILL.md             /pev-plan <request>        Opus, high
  pev-execute/SKILL.md          /pev-execute <issue>       per issue header
  pev-verify/SKILL.md           /pev-verify <issue>        Opus, high
  pev-epic-close/SKILL.md       /pev-epic-close <epic>     Opus, high
AGENTS-BLOCK.md                 append to AGENTS.md, then delete this file
ASSUMPTIONS.md                  what this rests on, evidence grades, how to port it
```

Everything is prefixed `pev-` so it never collides with a bundled skill. Claude Code ships its own `/verify`; `/pev-verify` is unambiguous.

## Install

1. Copy `.claude/skills/` into your repository root.
2. Append the fenced block from `AGENTS-BLOCK.md` to your `AGENTS.md` (create it if absent). Read the gotchas in that file first - **Claude Code ignores `AGENTS.md` if a `CLAUDE.md` exists at or above your working directory.**
3. Keep `ASSUMPTIONS.md` somewhere you will find it in six months.
4. Commit, then run `/pev-bootstrap`.
5. Confirm with `/skill`. Note that a directly-read `AGENTS.md` does **not** show under `/context` or `/memory`.

The five workflow skills carry `disable-model-invocation: true` - they have side effects and only fire when you invoke them by name. `pev-methodology` is model-invocable so Claude can read the doctrine when it genuinely needs it.

## Daily use

**Everything starts at `/pev-triage`.** It classifies the request and runs the right skill in the same session.

```
/pev-triage
     ├── advise  → /pev-advise ──→ findings + recommendation, nothing changed
     │                             (say go → re-triage in a NEW session)
     ├── simple  → /pev-simple ──→ PR → merge
     └── plan    → /pev-plan ────→ issues
                        ├── /pev-execute N  (one fresh chat per issue)
                        └── /pev-verify N   (fresh chat, Opus)
                                → merge → /pev-epic-close E
```

| When | Do |
|---|---|
| Any request | `/pev-triage <what you want>` in a fresh Opus chat |
| Triage said advise | It already ran `/pev-advise`. Read it. If you want it done, start a **new** session and triage the recommendation |
| Triage said simple | It already ran `/pev-simple`. Confirm the overview, review the PR |
| Triage said plan | It already ran `/pev-plan`. Answer the interview, approve |
| Each issue created | Fresh chat, `/pev-execute <N>`, model and effort per the issue header |
| After each issue | Fresh chat, `/pev-verify <N>`, Opus high |
| PASS | Merge |
| FAIL | Back to the execution chat, fix BLOCKER and MAJOR only |
| Last issue merged | Fresh chat, `/pev-epic-close <epic>`, Opus high |

## Routing

Three destinations, first match wins.

**`/pev-advise`** — the deliverable is information, not a change. Reviews, assessments, "should we", "what would you recommend", anything about a platform or config you would be assessing rather than editing. The tell: if the work were done and nothing changed, would you be satisfied? "Recommend changes to our hosting config" is advice, despite the word "changes". It writes nothing and ends by saying whether doing it looks like simple or plan work.

**`/pev-simple`** — a change where all five hold: bounded, no user decisions, no blast-radius surface, reversible by one PR revert, one sitting. Not code-only: README edits, config tweaks, dependency bumps and chores belong here too. Verification is proportionate — a docs change gets no ceremony, a code change gets the gate.

**`/pev-plan`** — everything else.

Condition three is the one that catches people. "Add a CSV export" clears the other four and fails that one the moment you ask who consumes the file.

## Skip conditions

`/pev-plan` refuses to run if `verify.sh` is missing or red. That is intentional. If your repo cannot produce a green gate today, your first epic is "make the check green".

## GitHub access

`GITHUB.md` defines the eight operations PEV needs and binds each to the GitHub MCP server or the `gh` CLI. Tool names move between MCP server versions, so every skill discovers what it actually has before calling. If both are available, either works.

Nested issues in the GitHub UI need an attach-child and a list-children operation. The MCP server has both; `gh` has them from v2.94.0. Without them, `/pev-plan` falls back to a flat task list and says so.

## Not Claude-exclusive

The workflow, the artifacts, and the rule block are portable. The SKILL.md packaging, effort levels, and model table are Claude Code specific. `ASSUMPTIONS.md` says which is which and how to port the rest.
