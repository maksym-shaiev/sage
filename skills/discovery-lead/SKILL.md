---
name: discovery-lead
description: Orchestrate the complete SAGE Discovery phase end to end — from a raw PRD to an approved architecture spec, ADR(s), and scope-of-work, ready for jira-backlog-builder. Sequences discover-prd, adr-writer, architecture-writer, and scope-mapper, and self-checks the full artifact set against the SAGE Constitution before declaring Discovery complete. Use at the start of a new initiative, or as the single entry point instead of invoking the four skills individually in order.
license: MIT
metadata:
  audience: developers, ai-agents
  workflow: discovery
  layer: core
---

## Governance

This skill operates under the [SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md)
in full — it is the skill responsible for the *whole* Discovery phase, not one article.
It does not re-implement any rule; it sequences the four skills that already implement
D1–D8, and adds one thing none of them can check alone: that the **complete** artifact
set (brief, ADRs, architecture, scope) is internally consistent and satisfies the
Discovery Gate (Article IV) together, not just skill-by-skill.

## What I do

Run the full Discovery pipeline from a raw PRD to an approved, gate-passed artifact set:

```
1. discover-prd            → Discovery Brief (requirement inventory, contract,
                              gap report, Decisions & Assumptions Register)
                            ── HUMAN GATE (Article IV / D7) ──
2. adr-writer               → one ADR per brief-flagged decision (`ADR required: yes`)
3. architecture-writer      → architecture spec (Article V, all 8 dimensions)
   ⟲ if architecture surfaces a new decision → back to step 2, then resume step 3
4. scope-mapper              → scope-of-work (may surface its own late ADRs — see
                              "Ownership boundary" below)
5. Handoff Criteria self-check across the complete artifact set
```

Note this is a **two-level orchestration**: `discover-prd` is itself an orchestrator
(it sequences `api-contract-extractor`, `gap-analyzer`, and its own Figma inventory step
internally). This skill sequences `discover-prd` as a single step, then continues past
where `discover-prd` stops.

## When to use me

- Starting a new initiative from a raw PRD, with none of the Discovery artifacts
  written yet
- You want one entry point instead of invoking `discover-prd`, `adr-writer`,
  `architecture-writer`, and `scope-mapper` individually in the right order
- Not needed if you are resuming partway through — e.g. the brief is already approved
  and you only need architecture and scope; in that case invoke the remaining skills
  directly (this skill still documents the correct order to follow)

## Pre-conditions

- A PRD URL, Jira epic, or written requirements document is available
- Either a live OpenAPI/Swagger endpoint, a service codebase, or (Design Mode) nothing
  but the PRD itself — `api-contract-extractor` handles all three, see its Extract
  Mode / Design Mode split

## Steps

### Step 1 — Discovery Brief (`discover-prd`)

Invoke `discover-prd` in full. It runs its own Steps 1–6 internally (PRD read, contract
extraction, Figma inventory, gap analysis, ADR-candidate flagging, brief assembly) and
enforces its own human gate — **do not bypass or duplicate that gate here.** Do not
proceed to Step 2 until `discover-prd` reports the brief is approved (its Step 6).

### Step 2 — ADRs for brief-flagged decisions (`adr-writer`)

For every open question in the brief's Open Questions table marked `ADR required: yes`,
invoke `adr-writer` once. Do this **before** architecture — the architecture spec should
cite settled decisions, not make them.

**Ownership boundary:** this step covers only decisions the brief already flagged. It
does not cover decisions that surface later, during architecture drafting (Step 3) or
scoping (Step 4) — those are each step's own responsibility (see Steps 3 and 4).

### Step 3 — Architecture spec (`architecture-writer`)

Invoke `architecture-writer`, passing the approved brief and the ADRs from Step 2.

**If architecture drafting surfaces a new decision** (a real ≥2-option choice not
covered by any existing ADR — most often in Integration Points or Auth Model, per
`architecture-writer`'s own notes on what's commonly missed): stop, invoke `adr-writer`
for it, then resume architecture drafting. This is the constitution's "ADRs →
architecture, iterating" sequence — do not silently pick an option and write it into
the architecture as settled (D1/D2).

### Step 4 — Scope of work (`scope-mapper`)

Invoke `scope-mapper`, passing the approved brief, the full ADR set (Steps 2 and any
from Step 3's loop), and the architecture spec.

**Ownership boundary:** `scope-mapper`'s own Step 3b remains the mechanism for
decisions that surface for the first time *during scoping* (e.g. a scope-boundary call
about what ships in v1). This skill does not duplicate that — it only ensures Steps 2–3
have already handled everything the brief and architecture already knew about.

### Step 5 — Handoff Criteria self-check

Before declaring Discovery complete, check the **complete** artifact set — not each
artifact individually, since each skill already self-checks its own output:

- [ ] Every scope item's `**Decision:**` field (if present) links to an ADR that exists
      on disk, and that ADR's See Also links back to the scope item (bidirectional —
      `adr-writer`'s own rule, verified here across the *whole* set)
- [ ] Every ADR referenced anywhere (brief, architecture, scope) exists and is `Status:
      Accepted`, not left at `Draft`
- [ ] The architecture spec's Integration Points table and the scope-of-work's
      `**Lanes:**` fields agree on which lane owns which endpoint
- [ ] No endpoint in the contract is `status: new`/`extension` with no owning scope
      item anywhere in scope-of-work (an orphaned proposed endpoint)
- [ ] The Decisions & Assumptions Register (in the brief) has zero entries still in the
      raw Assumed state (D7) — re-confirm this at the *end*, since architecture or
      scoping may have surfaced and then resolved new entries since Step 1's gate
- [ ] A human has reviewed and approved the complete set (brief + ADRs + architecture +
      scope) as a whole, not only the brief in isolation

If any box is unchecked, Discovery is not complete — resolve it before treating the
initiative as ready for `jira-backlog-builder`. This checklist is advisory (no
deterministic audit exists yet) but should be treated as a hard gate on this skill's
own output.

## Refuse conditions

- `discover-prd`'s human gate has not passed — do not proceed to Step 2
- An ADR is still `Status: Draft` when architecture or scope references it as settled
- The Step 5 self-check has an unresolved item — do not declare Discovery complete

## Output

The complete Discovery artifact set for the initiative:
- Discovery Brief (`discover-prd`)
- ADR(s) (`adr-writer`)
- Architecture spec (`architecture-writer`)
- Scope-of-work (`scope-mapper`)

Ready for `jira-backlog-builder`.

## Reference output

`../../reference/example-initiative/` — the complete four-artifact set this skill
produces, in the order this skill produces it. Useful as an end-to-end trace: read
`discovery-brief.md` → `decisions/ADR-001-dedicated-favorite-entity.md` →
`architecture.md` → `scope-of-work.md` in that order to see the sequence this skill
describes actually followed.

## See Also

- `../../constitution/SAGE-CONSTITUTION.md` — the full constitution this skill's
  sequence exists to satisfy end to end
- `skills/discover-prd/SKILL.md` — Step 1; itself an orchestrator (see its own See Also)
- `skills/adr-writer/SKILL.md` — Step 2 and the Step 3 loop
- `skills/architecture-writer/SKILL.md` — Step 3
- `skills/scope-mapper/SKILL.md` — Step 4
- `skills/jira-backlog-builder/SKILL.md` — the next phase, entered only after this
  skill's Step 5 self-check passes
- `../../vendor-shells/` — optional agent shells (OpenCode, Claude Code) that give this
  skill an `@`-mention entry point and an isolated context window; see `INSTALL.md`
