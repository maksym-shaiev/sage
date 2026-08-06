# Vendor Tooling Guide
Type: Guide
Status: Draft
Updated: 2026-06-12
Description: How to generate thin, optional, vendor-specific efficiency shells (opencode,
  Claude Code, GitHub Copilot) from the agnostic core skills, without moving any logic out
  of the core.
---

## Overview

The agnostic core (skills, guides, templates) produces correct results in any agent. This
guide explains how to generate **thin vendor shells** that make execution more efficient on
a specific tool. Shells contain **no logic** — they only point at a core skill and, where
the vendor supports it, add context isolation or parallelism.

Generate shells **on demand**, per vendor, when that vendor is actually used. A missing or
stale shell loses only the efficiency gain, never correctness.

## Principles

1. **No logic in shells.** A shell loads/invokes a skill and nothing more.
2. **Derivable + disposable.** Regenerate from the core at any time; the core wins on conflict.
3. **Equivalence is the contract.** Shell output must equal core output (same specs, same
   Jira tasks). Verify before relying on a shell.
4. **Efficiency is concentrated in read-isolation subagents.** Commands save a routing turn;
   parallelism saves wall-clock time (not tokens). Do not overclaim.

## Capability mapping

| Core artifact | opencode | Claude Code | GitHub Copilot |
|---|---|---|---|
| Skill (`SKILL.md`) | `.agents/skills/<n>/SKILL.md` or `.opencode/skill` (native) | `.claude/skills/<n>` (native Agent Skill) | `.github/` instructions/prompt file (closest; no native skill) |
| Command | `.opencode/command/<n>.md` (`/n`) | `.claude/commands/<n>.md` (`/n`) | `.github/prompts/<n>.prompt.md` |
| Subagent (context isolation) | `.opencode/agents/<n>.md` + built-in `explore` | `.claude/agents/<n>.md` | none — degrade to prompt + MCP |
| MCP integration | `opencode.json` `mcp` block | `.mcp.json` | VS Code `mcp.json` / Copilot MCP |

Notes:
- **opencode** and **Claude Code** both natively support skills, commands, and subagents —
  full efficiency layer available.
- **GitHub Copilot** has prompt files + MCP but **no subagent concept**, so its efficiency
  layer is partial; it falls back to the core most heavily. Treat Copilot support as a
  deferred, on-request target — build a shell for it when a consuming repo actually
  needs one, not speculatively.

## Generation recipes

### Command shell (any vendor)

A command is a thin pointer that skips the intent-routing turn.

opencode (`.opencode/command/<name>.md`):
```markdown
---
description: <one line>; thin shell over the <skill> skill
agent: build
---
Load and follow the `<skill>` skill. This is an efficiency-only shell; all logic lives in
the skill. Delegate heavy reads to the built-in `explore` subagent for context isolation.
$ARGUMENTS
```

Claude Code (`.claude/commands/<name>.md`): same body; Claude resolves `$ARGUMENTS`
similarly and can delegate to its own subagents.

Copilot (`.github/prompts/<name>.prompt.md`): same body minus subagent delegation
(unsupported); relies on MCP + the skill instructions only.

### Subagent shell (opencode / Claude Code only)

Use **only** for context isolation on heavy reads or large bulk ops. The subagent loads the
skill and runs it in a child context, returning a distilled result.

opencode (`.opencode/agents/<name>.md`):
```markdown
---
description: Runs the <skill> skill in isolation for context efficiency
mode: subagent
---
Execute the `<skill>` skill. Return only the distilled artifacts (paths, summaries, keys),
not raw intermediate output.
```

Prefer reusing the **built-in `explore`** subagent for research reads rather than authoring
a custom one — it already provides read isolation.

### Primary-agent shell — `discovery-lead`

`discovery-lead` (the Discovery orchestrator) is the one skill this kit ships a ready-made
agent shell for, because it's the natural `@`-mention entry point for a whole initiative —
see `vendor-shells/`:

| Vendor | Shell | Mechanism |
|---|---|---|
| opencode | `vendor-shells/opencode/opencode.json.example` (preferred) or `vendor-shells/opencode/discovery-lead.md` | JSON form uses `{file:...}` — zero duplication. Markdown form is a pointer body. |
| Claude Code | `vendor-shells/claude-code/discovery-lead.md` | Pointer body only — Claude Code markdown agents have no `{file:...}` equivalent, and don't auto-discover `.agents/skills/` (see that shell's own note) |
| Copilot | none shipped | No subagent concept; build a prompt-file shell on demand if a team needs one (see Capability mapping above) |

Copy, don't hand-write — see `INSTALL.md` § 4 and each `vendor-shells/<vendor>/README.md`
for setup steps. None of the other seven skills ship a shell; invoke them by name.

### Worked example — `discover-prd`

| Vendor | Shell | Efficiency added |
|---|---|---|
| opencode | `.opencode/command/discover-prd.md` (exists in Glue as of this writing) + `explore` for reads | routing-turn skip + read isolation + parallelism |
| Claude Code | `.claude/commands/discover-prd.md` (mirror body) + Claude subagents | same |
| Copilot | `.github/prompts/discover-prd.prompt.md` | routing-turn skip only; reads run inline |

All three invoke the same `discover-prd` skill and produce the same Discovery Brief.

## Equivalence check

Before relying on a new shell, run the underlying skill directly once and compare outputs
on a known input — `reference/example-initiative/` is small enough to run end-to-end and
compare by hand: same specs, same scope items, same Jira task set. If they differ, the
shell has logic it shouldn't — move it back into the skill.

## See Also

- `README.md` — kit overview and consumption
- `skills/` — the core skills shells wrap
- `vendor-shells/` — the one ready-made shell (`discovery-lead`), canonical and versioned
- `INSTALL.md` § 4 — setup steps for the `discovery-lead` agent shell
