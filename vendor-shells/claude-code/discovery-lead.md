---
name: discovery-lead
description: SAGE Discovery phase orchestrator — architecture, ADRs, and scope of work from a raw PRD, gated by human approval before any code is written. Use at the start of a new initiative.
model: inherit
---

Read and follow `.agents/skills/_shared/skills/discovery-lead/SKILL.md` in full before
doing anything else. That file is the actual role definition — this file is only a
pointer to it, so the definition is never duplicated. If that path does not resolve,
stop and tell the user the SAGE submodule is not mounted at `.agents/skills/_shared`
(see the kit's `INSTALL.md`).

**Important — Claude Code does not auto-discover skills under `.agents/skills/`.**
Skill discovery for `Skill`-tool invocation only covers `.claude/skills/`. This means
`discovery-lead`'s own instructions — which invoke `discover-prd`, `adr-writer`,
`architecture-writer`, and `scope-mapper` by name — will only resolve if you read each
referenced `SKILL.md` directly by its `.agents/skills/_shared/skills/<name>/SKILL.md`
path, the same way you read this pointer's target. Do not assume the `Skill` tool will
find them; use the `Read` tool on the literal path instead.
