---
name: gap-analyzer
description: Cross-reference PRD requirements against a verified API contract to produce a coverage table and open-question list with technical evidence and H/M/L risk. Use after the contract is extracted, before scoping.
license: MIT
metadata:
  audience: developers
  workflow: discovery
  layer: core
---

## Governance

This skill operates under the [SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md).
**D2 (ambiguity → open question, never a plausible default) is a hard stop.** A gap
between the PRD and the contract is a finding to be recorded, not a decision this skill
makes on its own — every open question it produces feeds the Decisions & Assumptions
Register (Article III) and must carry risk level, owner, and evidence; none may be
resolved silently.

## What I do

Compare what the PRD requires against what the verified contract provides, and produce a
gap report: a coverage table (every requirement → covered / deferred / blocked) plus open
questions with technical evidence and risk levels.

## When to use me

- The API contract has been verified (see `api-contract-extractor`).
- The PRD requirement inventory is available.
- Before writing the scope of work.

## How to use me

1. **Enumerate every PRD requirement** (typically a requirements table). Include
   importance (Must / Even Better If).
2. **Map each to the contract** — is it fully supported, partially, or absent?
3. **Apply the coverage rule:** every Must requirement must end up mapped to a scope item,
   a documented deferral, or a blocked status. Nothing silently dropped.
4. **For each gap, write an open question** (`../../templates/open-question.md`), using
   zero-padded `OQ-NNN` numbering, with:
   - PRD requirement vs API/system reality
   - **Technical evidence** — cite the live OpenAPI (endpoint absent), codebase
     (migration that dropped a table), or free-form schema. Evidence, not opinion.
   - Risk level **High / Medium / Low** and what it blocks.
   - Options to propose to the owning team.
5. **Respect precedence** (`../../guides/source-of-truth-precedence.md`): codebase/live
   API over PRD for "what exists"; PRD over all for "what is wanted".
6. **Emit the gap report** (`../../templates/gap-report.md`): coverage table, open
   questions, deferred (Must, tracked elsewhere).

## Output

A gap report feeding the scope of work (deferrals, blocked items) and stakeholder comms
(the open questions with risk levels go into the VP email and Jira comment).

## Reference output

- `../../reference/example-initiative/discovery-brief.md` — conformant, small; its
  `## Gap Report` and `## Open Questions` sections show `OQ-NNN` zero-padded, with
  risk/owner/`ADR required` populated

## See Also

- `../../templates/gap-report.md`, `open-question.md`
- `../../guides/source-of-truth-precedence.md`
- `apps/quest-ic/glue/docs/specs/incentives/qims-api-contract.md` — **historical**
  reference open questions; predates zero-padded `OQ-NNN` (uses unpadded `OQ-1`..`OQ-3`)
  — do not copy that numbering
- `scope-mapper` skill (consumes deferrals/blocked items)
