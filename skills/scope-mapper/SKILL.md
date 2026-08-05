---
name: scope-mapper
description: Map PRD requirements plus a verified API contract and Figma inventory into a phased scope-of-work spec, one item per requirement with an API contract block and Figma reference. Use after discovery and gap analysis, before creating a Jira backlog.
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: scoping
  layer: core
---

## What I do

Transform an approved Discovery Brief into a phased scope-of-work spec. Each PRD
requirement becomes one scope item with an API contract block, Figma reference, and
implementation note. The output feeds `jira-backlog-builder` directly.

## When to use me

- After `discover-prd` has completed and the Discovery Brief is human-approved
- Before creating any Jira tasks
- When you have: PRD requirement inventory, verified API contract, gap report, Figma inventory

## Pre-conditions

- Discovery Brief approved by a human (no unacknowledged High-risk open questions)
- PRD requirement inventory (Must / Even Better If) available
- Verified API contract available (from `api-contract-extractor`)
- Gap report with open questions available (from `gap-analyzer`)

## Inputs

Read these from the Discovery Brief or ask the human if missing:

| Input | Source |
|---|---|
| PRD requirement inventory | `discover-prd` Step 1 output |
| Verified API contract | `api-contract-extractor` output |
| Gap report + open questions | `gap-analyzer` output |
| Figma screen inventory | `discover-prd` Step 3 output |
| Target project key | Spec `Ticket:` header or ask human |
| Parent epic key | Spec overview or ask human |

## Steps

### Step 1 — Check pre-conditions

If the gap report contains unresolved **High**-risk open questions that have not been
explicitly accepted by the human: list them and halt. Do not produce scope items for
requirements blocked by an unacknowledged High-risk gap.

### Step 2 — Phase the requirements

Group Must requirements into delivery phases (earliest value first):
- **Phase 1** — MVP / pilot-unblocking items; smallest shippable slice
- **Phase 2** — Core functionality
- **Phase 3** — Admin, reporting, extended features
- After all Must phases: Even Better If items
- Last: Blocked items (awaiting open question resolution)

Use the gap report and ADR decisions to determine phasing. When unclear, ask the human.

### Step 3 — Write scope items

One block per requirement, in phase order. Use this template (adapted from
`_shared/templates/scope-item.md`):

```
### {ITEM-ID} — {Title}

**PRD:** "{PRD user story or requirement verbatim}"

**Importance:** Must | Even Better If

**Notes:** {clarifying context from gap report or discovery}

{If blocked}: **Status: BLOCKED** — awaiting {OQ-ID}. No tasks until resolved.

**Decision:** {ADR link if a decision governs this item — omit if none}

**Figma:** [{screen name}]({figma url}?node-id={node-id})  ← omit for BE-only items

#### API contract

{METHOD} {/endpoint}
Auth: {scheme}
Request: {shape — field names and types}
Response {code}: {shape}

**Implementation note:** {free-form — e.g. "new endpoint", "extends existing GET /x",
"BE-only — no FE task", "FE-only — endpoint already exists"}
```

**Coverage rule (hard):** every Must requirement must end in one of:
- A scope item (with or without a Figma reference)
- A documented deferral (with reason and tracking reference)
- A blocked status (with OQ-ID)

Nothing is silently dropped. If a Must requirement cannot be mapped, record it as blocked
with the reason — do not omit it.

**API contract guidance:**
- Use the verified contract from `api-contract-extractor` — never invent endpoint paths,
  field names, or response shapes
- If a required endpoint does not yet exist in the API, mark it `(new endpoint — to be
  implemented)` and use the PRD's proposed shape with a note it is unverified
- Free-form fields (`additionalProperties: true`) stay marked UNVERIFIED until the owning
  team confirms expected keys

### Step 3b — Create ADRs for flagged candidates

For every open question marked `ADR required: yes` in the Discovery Brief (from
`discover-prd` Step 4b), invoke the `adr-writer` skill to produce the ADR now — before
writing scope items — because scope items reference their governing ADR in the
`**Decision:**` field.

Also create ADRs for any new decision points that emerge while writing scope items
(e.g. a scoping choice between two valid approaches that was not in the original gap report).

**For each ADR produced:**
1. Save to `docs/specs/{domain}/decisions/ADR-{NNN}-{slug}.md`
2. Note the ADR path — it will be referenced in the relevant scope item's `**Decision:**` field
3. Add it to `docs/specs/README.md`

**Sequence:** write all ADRs for a phase before writing the scope items for that phase,
so every scope item can reference its governing decision from the start.

### Step 4 — Append the estimation table

Using `_shared/guides/estimation-heuristics.md`:

```markdown
## Delivery Estimate

### Complexity per item

| ITEM-ID | Title | BE complexity | BE days | FE complexity | FE days |
|---|---|---|---|---|---|
| {ID} | {title} | {Low/Med/High} | {n} | {Low/Med/none} | {n} |

Complexity signals (BE):
- Low — standard CRUD, single model, established pattern: 0.5–1.5 days
- Medium — single new class, one upstream integration: 1–2 days
- Medium-High — multiple upstream calls or merged data: 2–3 days
- High — multi-step flow, unfamiliar/legacy codebase change: 3–5 days + 1-day spike

AI-assist factors applied: BE ~50–60% faster; FE ~25–30% faster vs. unassisted.

### Phase timeline

| Phase | Items | BE days | FE days | Phase duration |
|---|---|---|---|---|
| Phase 1 | {IDs} | {n} | {n} | max(BE, FE) |
| Phase 2 | {IDs} | {n} | {n} | max(BE, FE) |

Phase duration = max(BE days, FE days) — BE and FE run in parallel.
Total = sum of phase durations + 15% buffer.

**Total estimate: ~{n} weeks** (1 BE + 1 FE, AI-assisted, full-time).

> High-uncertainty items carry a 1-day spike before their estimate is committed.
> Blocked items are excluded until the blocking OQ resolves.
```

### Step 5 — Write the spec file

Save to `docs/specs/{domain}/scope-of-work.md` using the standard metadata header:

```
# {Initiative} — Scope of Works
Type: Spec
Ticket: {Jira epic key}
Status: Draft
Updated: {today YYYY-MM-DD}
Description: Phased scope of work for {initiative} — {n} items with API contracts and estimates.
---
```

Update `docs/specs/README.md` with the new entry.

## Refuse conditions

- Discovery Brief not yet human-approved — do not scope without approval
- Unacknowledged High-risk open questions — list them and halt (Step 1)
- Required API contract is unavailable — run `api-contract-extractor` first

## Output

`docs/specs/{domain}/scope-of-work.md` — the approved scope of work, ready for
`jira-backlog-builder`

## Reference outputs

- `docs/specs/incentives/scope-of-work.md` — QIMS integration (24 items, 3 phases)
- `docs/specs/survey/reminders/scope-of-work.md` — Survey Reminders (18 items, K/G/FE split)
- `docs/specs/survey/activities-v2-delivery-scope.md` — Activities V2 cross-team scope

## See Also

- `skills/discover-prd/SKILL.md` — produces the inputs this skill consumes
- `skills/jira-backlog-builder/SKILL.md` — consumes this skill's output
- `skills/gap-analyzer/SKILL.md` — gap report input
- `guides/source-of-truth-precedence.md`
- `guides/estimation-heuristics.md`
- `templates/scope-item.md`
