---
name: pev-methodology
description: The Plan/Execute/Verify working methodology - tiers, blast radius sweep, model and effort assignment, waves and file ownership, trust boundary, anti-drift rules. Read this when planning work, breaking work into GitHub issues, verifying a diff, or when asked how the PEV workflow operates.
---

# Plan / Execute / Verify

Opus plans and closes every decision. GitHub issues carry the plan. Sonnet executes one issue per fresh chat. A deterministic script decides pass or fail; a read-only verifier decides whether the right thing was built.

**Full doctrine is in `.claude/skills/pev-methodology/REFERENCE.md`. Read it before planning or verifying.**
**GitHub access is in `.claude/skills/pev-methodology/GITHUB.md`. Read it before any GitHub call.**

## The loop

**Everything starts at `/pev-triage`.** It classifies and routes in the same session.

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

| Gate | Chat | Model | Effort | Skill |
|---|---|---|---|---|
| W0 Bootstrap (once per repo) | own | Sonnet | med | `/pev-bootstrap` |
| **Entry** | own | Opus | med | `/pev-triage` |
| Investigate and recommend | same session as triage | Opus | high | `/pev-advise` |
| Small bounded change | same session as triage | Sonnet | low-high | `/pev-simple` |
| G0-G2 Triage, interview, plan | same session as triage | Opus | high | `/pev-plan` |
| G3 Issue creation | same as G2 | - | - | end of `/pev-plan` |
| G4 Execute | one per issue | per issue | per issue | `/pev-execute N` |
| G5 Verify | fresh | Opus | high | `/pev-verify N` |
| G6 Merge | human | - | - | - |
| G7 Epic close | fresh | Opus | high | `/pev-epic-close E` |

Two consecutive FAILs on one issue means the issue is wrong, not the executor. Back to planning.

## Unit of work

One issue = one chat = one branch = one PR = one verification pass. If an issue cannot be executed by a cold-start chat reading only the issue body plus the files it names, the issue is written wrong. More than roughly 30k tokens of reading to start means split it.

## Routing

Three destinations, first match wins.

- **`/pev-advise`** - the deliverable is information, not a change. The tell: if the work were done and nothing changed, would the user be satisfied? "Recommend changes to our hosting config" is advice, despite the word "changes".
- **`/pev-simple`** - a change where all five hold: bounded (every file nameable now), no user decisions, no blast-radius surface, reversible by one PR revert, one sitting. Code, docs, config and chores alike.
- **`/pev-plan`** - everything else.

One uncertain condition earns one question. Two or more route to planning.

Routing everything through the planner is how a methodology dies. So is planning nothing.

## Artifacts

```
init.sh                  start the env, idempotent
verify.sh                THE gate. one command, one exit code
.plan/11/criteria.json   acceptance criteria, source of truth
.plan/11/progress.md     append-only session log
```

`11` is the epic's issue number. Everything about an epic is keyed to it.

## How issues join to an epic

**GitHub access is defined in `.claude/skills/pev-methodology/GITHUB.md`.** Eight operations, bound to the GitHub MCP server or the `gh` CLI. Tool names move between versions, so every skill discovers before it calls.

| Mechanism | Who uses it | Available in |
|---|---|---|
| Sub-issue relation | GitHub UI: nesting under the epic, progress rollup, Relationships panel | nested mode |
| Task list in the epic body | humans, and the only visual grouping in flat mode | both |
| `**Epic:** #11` and `**Plan:** .plan/11/` in the child header | a cold-start execution chat | both |
| Label `epic-11` | a second independent source at close time | only if your binding can create labels |

**Nested mode** needs an attach-child operation and a list-children operation. The GitHub MCP server has both (`sub_issue_write`, `issue_read` method `get_sub_issues`); `gh` has them from v2.94.0. Without them you get **flat mode**: a task list and header text, no nesting in the UI. `/pev-plan` probes and says which it used.

The header line is the one that matters at execution time: an executor is given one issue number and nothing else, and must find `criteria.json` without guessing or reading the epic. Paths in a created issue are always literal, never placeholders.

At epic close, all three must agree. A mismatch means an issue was created outside the plan, or a criterion points at an issue that does not exist.

## Who writes what

| Artifact | Written by | Executor may edit? |
|---|---|---|
| `verify.sh` | bootstrap, once per repo | **No** |
| `criteria.json` | planner at G2 | only the `passes` field |
| issue `Verification:` command | planner | No |
| the test file | executor | Yes, that is its job |

The executor writes tests, not the gate. A model that can edit its own gate has no gate.
