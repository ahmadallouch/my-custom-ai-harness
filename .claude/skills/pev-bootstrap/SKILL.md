---
name: pev-bootstrap
description: Bootstrap a repository for PEV agent-driven development by creating init.sh and verify.sh. Run once per repository, before the first plan.
disable-model-invocation: true
---

# Bootstrap (Wave 0)

Run once per repo. Sonnet, medium effort. `/pev-plan` refuses to proceed without this.

Produce exactly four things. Nothing else. No refactors, no cleanup, no README rewrite.

## 1. `init.sh` at the repo root

- Idempotent. Safe to run twice.
- Installs deps if missing, starts whatever a dev needs running (db, server), exits 0 when the app is actually usable.
- Prints the URLs and ports it exposed.
- If nothing needs starting, it says so and exits 0. Do not invent services.

## 2. `verify.sh` at the repo root - THE GATE

- `./verify.sh` with no args runs the full check: tests, typecheck, build, whatever this repo already has. Exit 0 = green.
- `./verify.sh <filter>` passes the argument THROUGH to the test runner as a filter. **It is a dispatcher, not a registry.** Do not hardcode a list of targets. Do not write a case statement of feature names. A future feature's target must exist the moment its test file is named, with no edit to this script. This is deliberate: nobody may edit the gate after today.
- Use the commands this repo ALREADY has. Do not add a test framework, do not add dependencies, do not write tests. If there is no test command, say so plainly and make verify.sh run whatever does exist (lint, typecheck, build). A weak gate that exists beats a perfect one that does not.
- `set -euo pipefail`. No swallowed failures.
- **A filter that matches zero tests is a FAIL, never a pass.** A typo in a filter must not read as a green gate. Runners disagree here: some already exit non-zero when nothing runs (pytest exits 5), some only if you leave their "pass with no tests" option off (Jest), and some exit 0 and print a notice (Go prints "no tests to run"). So:
  - Never enable a pass-with-no-tests flag.
  - If the runner already fails on zero matches, rely on that.
  - If it does not, capture the run's output, count the tests that actually ran, and exit 1 with `verify: filter '<filter>' matched no tests` when the count is zero.
  - Do not trust this list of runner behaviours. Prove it on this repo (see Rules).

Shape:

```bash
#!/usr/bin/env bash
set -euo pipefail
pnpm typecheck
pnpm test "${1:-}"
```

## 3. `.plan/README.md`

Three lines describing what `.plan/<epic>/criteria.json` and `progress.md` are for. Do not create an epic.

## 4. Report to the user

Max one paragraph and five bullets:
- what init.sh starts
- what verify.sh actually checks, and what it does NOT check
- anything stubbed or undetermined, prefixed with a warning marker on its own line
- the single highest-value thing missing from this repo's feedback loop
- whether `./verify.sh` currently exits 0 on a clean checkout, and the exit code of the zero-match probe

## Rules

- Do not write application code. Do not write tests. Do not fix failing tests.
- Do not add dependencies.
- Do not modify existing config beyond what the two scripts need.
- Do not speculate about commands you have not run. Run them.
- Run `./init.sh` and `./verify.sh` yourself before reporting. Paste the output.
- Prove the zero-match guard: run `./verify.sh zzz-no-such-test-zzz` and confirm it exits NON-zero. Paste the output and exit code. If it exits 0, the gate is broken; fix the script before reporting.
- If verify.sh cannot be made to exit 0 because the repo is genuinely broken, say so and stop. Do not weaken the script until it passes.

## Afterwards

Nobody edits `verify.sh` in the normal loop. The executor writes tests, not the gate. If an executor needs the gate changed - a missing stage, a new service, a filter that does not reach its tests - that is a stop condition, and the user re-runs `/pev-bootstrap` in its own chat.
