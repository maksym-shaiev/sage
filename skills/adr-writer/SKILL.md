---
name: adr-writer
description: Create Architecture Decision Records (ADRs) for any significant decision surfaced during Discovery or Delivery — technical, product, integration, configuration, or pattern decisions. Use when an open question is resolved by choosing between two or more real options, when a PRD requirement conflicts with codebase reality and a call must be made, or when a Product Owner, architect, or tech lead makes an explicit choice. Triggers on: "ADR", "decision record", "architecture decision", "we need to decide", "two options", "product decision", "scope decision".
license: MIT
compatibility: opencode
metadata:
  audience: developers, ai-agents
  workflow: discovery, delivery
  layer: core
---

## Governance

This skill operates under the [SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md).
It is the primary implementation of **D4 (every decision carries Decided-by, Decided-on,
and what it was decided against)** — every ADR this skill produces is a Confirmed entry
in the Decisions & Assumptions Register (Article III), and must record the decider, the
date, and the rejected alternative(s), not just the chosen option.

## What I do

Produce a well-structured ADR for every significant decision encountered during Discovery
or Delivery. An ADR is not only for technical choices — product scope decisions, UX
behaviour calls, integration contract choices, and ops/configuration approaches all
warrant an ADR when two or more real options existed.

## When to create an ADR

Create an ADR when **any** of the following are true:

- An open question (OQ-NNN) was resolved by choosing between ≥ 2 real options
- A PRD requirement conflicts with the codebase or API reality and a call was made to
  resolve the gap (adjust requirements, defer, or add new API surface)
- An integration contract choice was made — single vs multiple endpoints, where to apply
  business logic, which service owns a piece of data
- A scope boundary was decided — what ships in v1 vs deferred, what is adjusted to fit
  the existing architecture rather than extending it
- An ops/config approach was chosen — caller-specified vs pre-configured, where a value
  lives (DB, config, runtime input)
- A Product Owner, architect, or tech lead made an explicit call between options
- A design pattern was chosen over an equally valid alternative

**Do not create an ADR for:** implementation details with a single obvious approach,
naming choices, code style preferences, or decisions with no real alternative.

## ADR classification (Subtype)

Every ADR must declare a `Subtype:` in the metadata header:

| Subtype | When to use | Examples |
|---|---|---|
| `Architecture` | System structure, component ownership, data model design, service boundaries | No new reminder entity; resolve via Kato not IDF |
| `Product` | Scope, UX behaviour, PRD interpretation, feature boundary calls made by PO | Flat tab list across sendouts; adjust mocks to fit architecture |
| `Integration` | API design, endpoint structure, contract ownership, where logic lives across services | Single POST endpoint for immediate + scheduled; channel intersection in Glue |
| `Configuration` | Where values live — ops vs runtime vs caller-specified | Email template as panel-level ops config; login autokey from DB |
| `Pattern` | Which established code pattern to follow when alternatives exist | Use generic provider/processor chain vs custom State class |

A single ADR may span two subtypes (e.g. both `Architecture` and `Product`). In that case
use the primary driver as the Subtype.

## Metadata header

```
# {Decision title — imperative, e.g. "Use Existing Kato Sendout Architecture"}
Type: Decision
Subtype: {Architecture | Product | Integration | Configuration | Pattern}
Ticket: {Jira key — omit if not tied to a single ticket}
Status: Draft | Accepted | Superseded
Updated: {today YYYY-MM-DD}
Description: Why {approach X} was chosen over {approach Y} for {initiative/feature}.
Triggered-by: {OQ-NNN | "PRD conflict" | "discovery gap" | "PO escalation" — omit if not from a gap report}
Decided-by: {name and role of the person who made the call, e.g. "Jane Doe, Product Owner"}
Decided-on: {YYYY-MM-DD the decision was made — may differ from Updated}
---
```

`Decided-by` and `Decided-on` implement SAGE Constitution D4 (every decision carries who
decided, when, and what it was decided against). Both are mandatory — this ADR *is* the
Confirmed entry in the Decisions & Assumptions Register (Article III); without a named
decider and date it cannot be distinguished from an Assumed claim the agent wrote itself.

## Required sections

### Context

Background information only — no decision yet. Answer:
- What is the system or feature this decision is about?
- What constraint, gap, or conflict surfaced that required a decision?
- What sources of evidence were consulted (PRD, live API, codebase, PO conversation)?

Include a brief options table summarising the candidates before the full Options section:

```markdown
| Option | Approach |
|---|---|
| A — {name} | {one-line description} |
| B — {name} | {one-line description} |
```

### Problem

One paragraph. What must be decided, and why it cannot be left open.

### Options

One subsection per option. Each must include a pros/cons table:

```markdown
### Option A — {name}

| | |
|---|---|
| **Pro** | {pro} |
| **Pro** | {pro} |
| **Con** | {con} |
| **Con** | {con} |
```

For product decisions: include the source of the requirement being served (PRD section,
mock node ID, PO statement). For technical decisions: cite the live API, codebase path,
or data model as evidence — not opinion.

### Decision

State the chosen option explicitly. Then explain why — one paragraph. Explicitly name
which option(s) were rejected and why, even briefly — a decision record that only
justifies the winner does not satisfy D4.

For product decisions: quote the PO or decision-maker directly if available, with name
and date — this quote is the evidence behind the `Decided-by`/`Decided-on` header
fields, not a separate flourish. Example:
> *"Fundamentally we use what we have in sendouts, no big rewrite."*
> — Product Owner, 2026-06-26

For technical decisions: state the technical rationale (fewer hops, single source of
truth, fewer runtime dependencies, consistent with established pattern) and name the
option(s) it beat.

### Consequences

Numbered list. Cover all of:

1. **Positive outcomes** — what is gained
2. **Risks and limitations** — what could go wrong, what is lost
3. **PRD/mock adjustments required** — explicitly flag any requirement or design that must
   change as a result of this decision. Example: *"PRD adjustment required: the named
   reminder list in the mock must be redesigned — no independently configurable reminder
   records will exist."*
4. **Downstream obligations** — new contracts needed, ops tasks, future scope items this
   decision creates or blocks
5. **Deferral notes** — capabilities explicitly deferred and what would unlock them

### See Also

- Link to the OQ-NNN that triggered this ADR (gap report or Discovery Brief)
- Link to the scope item(s) this decision governs
- Link to related ADRs in the same initiative
- Link to the relevant spec (architecture, integration, contract)
- Jira ticket URL

## File placement and naming

```
docs/specs/{domain}/decisions/ADR-{NNN}-{slug}.md
```

Where:
- `{NNN}` is a zero-padded sequence within the initiative (001, 002, …)
- `{slug}` is a short kebab-case description of the decision (not the chosen option)

Examples:
- `docs/specs/survey/reminders/decisions/ADR-001-no-new-reminder-entity.md`
- `docs/specs/decisions/member-resolve-via-kato.md` (cross-initiative decisions live at top level)

For initiatives with only 1–2 ADRs, a flat `docs/specs/decisions/` placement is acceptable.
For initiatives with ≥ 3 ADRs, use a dedicated subdirectory per initiative.

Update `docs/specs/README.md` with the new ADR entry under the relevant domain section.

## Scope item linkage

Every ADR that governs a scope item must be cross-referenced in both directions:

- In the ADR's See Also: link to `docs/specs/{domain}/scope-of-work.md ({ITEM-ID})`
- In the scope item's `**Decision:**` field: link to the ADR

## Quality checks before finalising

- [ ] Options table has ≥ 2 rows with real trade-offs — not a foregone conclusion
- [ ] `Decided-by` and `Decided-on` are populated with a real name/role and date (D4) —
      never left as a placeholder or inferred by the agent
- [ ] Decision section names the rejected alternative(s), not only the winner (D4)
- [ ] Decision cites authority (PO name + date, or technical evidence from API/codebase)
- [ ] Every PRD or mock impact is explicitly flagged as "PRD/mock adjustment required"
- [ ] Triggered-by field references the OQ-NNN or gap that prompted this decision
- [ ] File is linked from `docs/specs/README.md`
- [ ] Scope items that this decision governs have a `**Decision:**` back-link

## Reference ADRs

These are the canonical examples to follow:

| ADR | Subtype | Why it's a good example |
|---|---|---|
| `survey/reminders/decisions/ADR-001-no-new-reminder-entity.md` | Architecture + Product | PO direct quote; explicit PRD adjustment requirements per consequence |
| `survey/reminders/decisions/ADR-003-immediate-vs-scheduled.md` | Integration | Clean options table; downstream UX/SPA consequences numbered precisely |
| `decisions/member-resolve-via-kato.md` | Integration | Technical evidence cited (hop count, runtime dependency); contract spec included |
| `survey/reminders/decisions/ADR-004-reminder-email-template-config.md` | Configuration | Ops dependency consequence; graceful degradation fallback documented |
| `survey/reminders/decisions/ADR-005-multi-sendout-send-groups.md` | Product | PO direct quote; "consistent with existing model" rationale; FE obligation in consequences |

## See Also

- `constitution/SAGE-CONSTITUTION.md` — Article II (D4: decision provenance), Article
  III (this ADR is the Confirmed entry graduating an Assumed/Open register row)
- `skills/discover-prd/SKILL.md` — Step 4b surfaces ADR candidates during Discovery
- `skills/scope-mapper/SKILL.md` — Step 3b creates ADRs for flagged candidates
- `templates/open-question.md` — OQ format; OQ-NNN → ADR is the standard resolution path
