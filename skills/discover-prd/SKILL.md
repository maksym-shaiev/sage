---
name: discover-prd
description: Orchestrate the discovery phase of the SDD workflow — read a PRD plus linked sources, verify the API contract, inventory Figma, run gap analysis, and produce a discovery brief. Use at the start of a new integration. Delegates heavy reads to read-isolation subagents.
license: MIT
metadata:
  audience: developers
  workflow: discovery
  layer: core
---

## Governance

This skill operates under the [SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md).
Discovery rules D1–D8 apply; **D1, D2, and D7 are hard stops** — halt and resolve with
the human before proceeding rather than guessing past them. Every claim written into
the Discovery Brief carries a provenance marker (Article III: Verified / Confirmed) or
is lifted into the Decisions & Assumptions Register as Assumed/Open — never written as
settled fact with neither.

## What I do

Orchestrate the full SDD discovery pipeline from raw inputs (PRD, API, Figma) to an
approved Discovery Brief. Each step delegates heavy parsing to subagents for context
isolation. The Discovery Brief is the handoff to `scope-mapper`.

## When to use me

- Starting a new integration or feature initiative from a PRD or Jira epic
- Before any code is written and before a scope of work exists
- When you have a PRD (Confluence/Jira/doc) and a live API or service codebase to integrate against

## Pre-conditions

- A PRD URL, Jira epic, or written requirements document is available
- Either a live OpenAPI/Swagger endpoint OR a service codebase path is accessible

## Steps

### Step 1 — Read the PRD

Delegate to the built-in `explore` subagent (or equivalent read-isolation context) for
large Confluence pages or Jira epics — do not inline raw doc content into the main thread.

Extract and return:
- **Requirement inventory** — every stated requirement, labelled Must or Even Better If
- **Stakeholder intent** — the business goal in one sentence
- **Figma link(s)** — all design references mentioned in the PRD
- **Integration target** — which service/API is being integrated against

If the PRD is ambiguous about Must vs Even Better If, ask the human before continuing.

### Step 2 — Extract the API contract

Invoke the `api-contract-extractor` skill, passing:
- The live OpenAPI/Swagger URL (preferred — staging environment if available)
- Or the service codebase path if no live spec exists
- The requirement inventory from Step 1 — needed even when a live contract exists,
  because most initiatives need endpoints that don't exist yet. The skill runs Extract
  Mode against what's already live and Design Mode against what the PRD implies but the
  API doesn't have; both modes usually apply within the same initiative.

**Source-of-truth precedence (enforce always):**
1. Live API contract — what callers actually integrate against
2. Service codebase — authoritative for behaviour OpenAPI cannot express (free-form fields, auth guards, idempotency)
3. PRD / Confluence — authoritative only for *what is wanted*, never for *what currently exists*

A gap between PRD and live API is a finding, not an error in either source.

### Step 3 — Inventory Figma

For each Figma link from Step 1, enumerate:
- Screen names and node IDs
- Which PRD requirements map to which screens
- Any interactions or states visible in the design

Note: if no Figma link exists (BE-only initiative), skip this step and record "N/A — no
design reference."

This step runs inline until a dedicated `figma-inventory` skill is available.

### Step 4 — Run gap analysis

Invoke the `gap-analyzer` skill, passing:
- The PRD requirement inventory from Step 1
- The verified API contract from Step 2

Produce:
- **Coverage table** — every Must requirement mapped to: covered / partially covered / absent
- **Open questions** (OQ-NNN) — each with technical evidence, risk level (High / Medium / Low),
  options to propose, and the owning team

Nothing silently dropped: every Must requirement must end up covered, deferred, or blocked.

### Step 4b — Surface ADR candidates

For every open question (OQ-NNN) in the gap report, ask: "Was this resolved — or does it
resolve — by choosing between ≥ 2 real options?"

If yes, flag it as an ADR candidate. Add an `ADR required` column to the open questions
table in the Discovery Brief:

| OQ-ID | Question | Risk | Owner | ADR required |
|---|---|---|---|---|
| OQ-001 | ... | High | Kato team | Yes — Architecture |
| OQ-002 | ... | Medium | PO | Yes — Product |
| OQ-003 | ... | Low | — | No |

Also flag ADR candidates that arise from sources other than the gap report:
- PRD requirements that conflict with codebase reality and required a scope or design call
- Any explicit PO, architect, or tech lead decision made during discovery conversations
- Integration contract choices (endpoint structure, logic ownership, data ownership)

ADRs are written in `scope-mapper` Step 3b — not here. This step only identifies and
flags candidates so the Discovery Brief is complete.

### Step 5 — Assemble the Discovery Brief

Produce a single summary document containing:

```
# {Initiative} — Discovery Brief

## PRD Summary
{One paragraph: what the initiative is, business goal, stakeholders}

## Requirement Inventory
{Table: ID | Requirement | Importance | Figma screen}

## Verified API Contract
Reference: {path to contract spec, e.g. docs/specs/{domain}/{name}-api-contract.md}
Key findings: {endpoints available, auth model, free-form fields, state enums}

## Figma Screen Inventory
{Table: Screen | Figma node | PRD requirements covered}

## Gap Report
Reference: {path to gap report or inline if short}

## Open Questions
{OQ-NNN table with risk level, owning team, status}

## Decisions & Assumptions Register
{Per SAGE Constitution Article III. One row per open question or pending decision:}

| ID | Statement | State | Evidence / Decided-by (+date) | Blocks | Resolution |
|---|---|---|---|---|---|
| OQ-001 | {claim or gap} | Assumed → Open | — | {ITEM-ID or "—"} | pending |
| OQ-002 | {claim} | Confirmed | {name/role} — {date} | — | inline (no ADR) |

{Every row MUST be Verified, Confirmed, or Open-and-blocking. Zero rows may remain
Assumed with nothing blocking them at the time this brief is presented for approval —
see Step 6.}

## Handoff note
Ready for scope-mapper once every register entry is Verified, Confirmed, or
Open-and-blocking (D7). {Approved by: {name}, {date}, against this version.}
```

Save to `docs/specs/{domain}/discovery-brief.md` (or agreed path).

### Step 6 — Human gate

Present the Discovery Brief. Before presenting, verify the Decisions & Assumptions
Register: every entry is Verified, Confirmed, or explicitly Open-and-blocking its scope
item (Article IV, condition 1–2). If any entry is Assumed with nothing blocking it,
this is a **hard stop (D7)** — resolve it (evidence, a human decision, or an explicit
block) before presenting.

Do NOT proceed to `scope-mapper` until the human explicitly approves. Record the
approval directly in the Handoff note: who approved, on what date, against which
version of this brief.

## Refuse conditions

- PRD URL or doc is not accessible — ask the human to provide it
- No live OpenAPI URL and no codebase path — cannot verify the contract; halt and request
- The Decisions & Assumptions Register contains an Assumed entry with nothing blocking
  it — hard stop (D7); resolve before presenting for approval
- Human gate not passed — do not invoke `scope-mapper` without explicit, recorded approval

## Output

`docs/specs/{domain}/discovery-brief.md` — the approved handoff document for `scope-mapper`

## See Also

- `constitution/SAGE-CONSTITUTION.md` — Article II (Discovery rules), Article III
  (provenance model + Decisions & Assumptions Register), Article IV (the gate this
  skill's Step 6 implements)
- `skills/api-contract-extractor/SKILL.md` — Step 2
- `skills/gap-analyzer/SKILL.md` — Step 4
- `skills/scope-mapper/SKILL.md` — next step after human gate
- `guides/source-of-truth-precedence.md` — precedence rule applied in Step 2
- `templates/open-question.md` — OQ format used in Step 4
