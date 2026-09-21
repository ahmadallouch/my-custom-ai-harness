# my-custom-ai-harness

PEV (Plan / Execute / Verify): Claude Code skills for planning in GitHub issues, executing one issue per chat, and verifying with a script gate plus a read-only reviewer.

## Install

1. Copy `.claude/skills/` into your repo root.
2. Append the block in `AGENTS-BLOCK.md` to your `AGENTS.md`.
3. Run `/pev-bootstrap`, then start every request with `/pev-triage`.

Details: [USAGE.md](USAGE.md). Assumptions: [ASSUMPTIONS.md](ASSUMPTIONS.md).
