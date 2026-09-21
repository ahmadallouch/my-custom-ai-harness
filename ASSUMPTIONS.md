# Assumptions, evidence, and portability

Everything in this pack is a working hypothesis, not a law. This file records what it rests on so you can tell which parts to keep when the ground shifts.

## The short version

The **workflow** is model-agnostic. Plan, break into issues, execute one at a time against a deterministic gate, verify independently, close the epic. That is ordinary software engineering discipline and it predates all of this.

The **calibration** is not. Effort levels, the specific anti-drift wording, the model assignment table, and the CLAUDE.md/AGENTS.md rule shape were all derived from and tested against Claude models on Claude Code. Some of it transfers, some of it is tuning for behaviour a different model may not have.

## Evidence grades

| Claim | Grade | Source |
|---|---|---|
| A runnable check is the highest-leverage thing you can give a coding agent | Strong | Anthropic Claude Code docs; widely replicated in practice |
| Context degradation as context fills; short focused sessions beat long ones | Strong | Anthropic Claude Code docs |
| Negative constraints help in rule files, positive directives hurt | Medium | Controlled study, 679 rule files, 5,000+ runs on SWE-bench Verified, **Claude Opus 4.6 only**, authors explicit about scope |
| Rule-file gains are largely content-independent (context priming) | Medium | Same study. Counterintuitive; treat as a reason not to over-invest, not as licence to write nonsense |
| Agents one-shot, run out of context mid-feature, and declare victory early | Medium | Anthropic long-running-agent harness research, built on a claude.ai-clone task |
| JSON resists inappropriate model edits better than Markdown | Medium | Same research, stated as an experimental finding |
| Agents mark features done on unit tests while the feature is broken end to end | Medium | Same research |
| The lethal trifecta makes issue-driven agent workflows exploitable | Strong | Willison's framing; multiple production CVEs, including one GitHub issue payload that broke Claude Code, Gemini CLI and Copilot Agent simultaneously |
| AI-assisted development can be net slower while feeling faster | Medium | METR RCT, 16 devs, 246 tasks, early-2025 tooling. METR published newer data in Feb 2026; the 19% figure is historical. The calibration gap is the durable part |
| Current Opus over-explores at high effort and over-spawns subagents | Medium | Anthropic prompting docs, model-specific and version-specific |
| Effort levels as low/medium/high/max | Weak | Product surface. Names and behaviour change between releases |
| GitHub MCP tool names (`issue_write`, `issue_read`, `sub_issue_write`) | Weak | GitHub consolidated its issue tools in late 2025 and may again. GitHub's own guidance is that individual tool names change between versions and toolsets are the stable surface. The skills discover rather than assume |
| `gh` sub-issue and dependency flags (`--parent`, `--blocked-by`) | Weak | GitHub CLI v2.94.0, changelog June 2026. Flag names and JSON field names may move |

**"Weak" does not mean wrong.** It means recheck before relying on it.

## The routing conditions are mine

The five `/pev-simple` conditions and the advise-vs-change tell are judgment, not measurement. They encode one belief worth stating: the expensive mistake is not misrouting a small job into planning, it is misrouting a large one out of it. The tie-break leans toward planning for that reason.

The cost of that lean is friction on genuinely small work, which is the failure mode that kills methodologies. If you find yourself skipping triage because it over-routes, loosen condition 1 or 5 first — never condition 3.

## What is Claude-specific

- **SKILL.md format and the `/skill-name` invocation.** A Claude Code convention. The `disable-model-invocation: true` frontmatter has no equivalent elsewhere.
- **`.claude/skills/` discovery.** No other agent reads this path by default. Cursor has an opt-in that picks it up; most do not.
- **Effort levels and the model names** in the assignment table.
- **`AskUserQuestion`** in the planner's interview step. Other agents will just ask in prose, which works but does not batch as cleanly.
- **Subagent token multiplier (4-7x)** and the over-spawn warning.
- **The exact anti-overthinking phrasings** ("choose an approach and commit to it") were lifted from Anthropic's own prompting guidance for their models.

## What is portable

- The loop, the gates, and the unit of work.
- The blast radius sweep and tier system. This is just engineering judgment written down.
- `criteria.json`, `verify.sh`, `init.sh`, `progress.md`. Plain files. Any agent can read and run them.
- GitHub issues as the carrier. `GITHUB.md` binds the eight operations PEV needs to either the GitHub MCP server or the `gh` CLI; adding a third binding is one table row each.
- The trust boundary rules. The lethal trifecta is not model-specific; every agent reading untrusted text with repo access has it.
- The decision ledger with reversibility, and the measurement proxies.
- The rule block in `AGENTS-BLOCK.md` - `AGENTS.md` is read by Codex, Cursor, Amp, Gemini CLI, and Claude Code.

## Porting to another agent

1. `AGENTS.md` already works almost everywhere. Start there.
2. The six SKILL.md bodies are just prompts. Strip the YAML frontmatter and use them as whatever your agent calls a command, rule, or saved prompt:
   - **Codex / Amp** - paste as a prompt, or reference from `AGENTS.md`
   - **Cursor** - `.cursor/rules/` files, or a saved prompt
   - **Gemini CLI** - `GEMINI.md`, or set `context.fileName` to read `AGENTS.md`
   - **Anything else** - paste the body at the start of the session
3. Re-tune the model and effort table for whatever you are running. The structure survives; the values do not.
4. Drop the subagent guidance if your agent has no subagents. Nothing depends on it.
5. Keep `verify.sh` unchanged. It is the part that works regardless of what is calling it.

## A note on MCP vs CLI context cost

Anthropic's own guidance is that CLI tools are the most context-efficient way to reach an external service, because an MCP server's tool definitions load into context whether or not you use them. Context degradation is the root constraint this whole methodology is built around, so the tradeoff is real: MCP buys structured results and a capability the CLI may lack (sub-issues on older `gh`), and costs context on every session.

If you are on MCP, trimming to the `issues`, `repos` and `context` toolsets rather than loading everything is the cheap mitigation. `GITHUB.md` needs only eight operations, all in `issues` and `repos`.

## Recheck triggers

Revisit this pack when:

- Effort level names or defaults change
- A model generation ships with materially different verbosity, scope, or subagent behaviour
- Claude Code changes how `AGENTS.md` is discovered or how skills are invoked
- The GitHub MCP server consolidates or renames tools again, or gains a label-creation or issue-dependency tool
- Newer measurement replaces the METR figures
- Your own G5 FAIL rate sits near zero (verifier rubber-stamping) or over half (planner underspecifying)

Written September 2026 against Claude Code with Opus and Sonnet. If you are reading this a year later, assume the weak-grade rows have moved.
