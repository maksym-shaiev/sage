---
name: discover-prd
description: Orchestrate the discovery phase of the SDD workflow — read a PRD plus linked sources, verify the API contract, inventory Figma, run gap analysis, and produce a discovery brief. Use at the start of a new integration. Delegates heavy reads to read-isolation subagents.
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: discovery
  layer: core
---

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

## Handoff note
Ready for scope-mapper once open questions are resolved or risk is accepted.
```

Save to `docs/specs/{domain}/discovery-brief.md` (or agreed path).

### Step 6 — Human gate

Present the Discovery Brief. Do NOT proceed to `scope-mapper` until the human
explicitly approves. Record which High-risk open questions are accepted vs. pending.

## Refuse conditions

- PRD URL or doc is not accessible — ask the human to provide it
- No live OpenAPI URL and no codebase path — cannot verify the contract; halt and request
- Human gate not passed — do not invoke `scope-mapper` without explicit approval

## Output

`docs/specs/{domain}/discovery-brief.md` — the approved handoff document for `scope-mapper`

## See Also

- `skills/api-contract-extractor/SKILL.md` — Step 2
- `skills/gap-analyzer/SKILL.md` — Step 4
- `skills/scope-mapper/SKILL.md` — next step after human gate
- `guides/source-of-truth-precedence.md` — precedence rule applied in Step 2
- `templates/open-question.md` — OQ format used in Step 4
