# Source-of-Truth Precedence
Type: Guide
Status: Draft
Updated: 2026-06-12
Description: The precedence rule for resolving conflicts between a live API, a service
  codebase, and a PRD/Confluence document during discovery and contract extraction.
---

## Overview

During discovery, the same fact (an endpoint, a field, a capability) is often described in
multiple places that disagree. This guide defines the precedence order and how to handle
conflicts so that specs and contracts reflect reality, not stale documentation.

## Precedence order

When sources conflict, trust them in this order (highest first):

1. **Live API contract** — the running service's OpenAPI/Swagger (preferably the target
   environment, e.g. staging). This is what callers actually integrate against.
2. **Service codebase** — routes, schemas, migrations in the service repo. Authoritative
   for behaviour the OpenAPI does not express (e.g. free-form object keys, auth guards),
   but secondary to the live contract for endpoint shape.
3. **PRD / Confluence / design docs** — intent and requirements. Authoritative for *what is
   wanted*, never for *what currently exists*.

Corollary: the PRD defines the target; the live API defines the present. A gap between them
is a finding, not an error in either.

## Handling conflicts

- **Live API vs PRD** — if the PRD requires a capability the live API lacks, record an open
  question with technical evidence and a risk level (see `../templates/open-question.md`). Do
  not assume the API will gain it; do not silently drop the requirement.
- **Codebase vs PRD** — if the codebase proves a capability was removed (e.g. a migration
  dropping a table), the removal wins over the PRD's assumption. Flag the PRD as stale.
- **Live API vs codebase** — for endpoint shape, the live contract wins. For undocumented
  internals (free-form config objects, idempotency behaviour), the codebase wins. If the
  codebase is in a different repo than the caller, read the service repo, not the caller.
- **Undocumented free-form fields** — when a schema declares an object as free-form
  (`additionalProperties: true`, no declared keys), treat its keys as UNVERIFIED. Propose
  PRD-derived keys but mark them pending confirmation from the owning team.

## Worked example (synthetic)

- A PRD requires a favorite-limit rule; no live endpoint or evidence specifies a value.
  Precedence does not resolve this one — there is no live/codebase source to defer to,
  only silence. That is recorded as an open question (`OQ-001`) and left open, not
  defaulted — see `../reference/example-initiative/discovery-brief.md`.
- A PRD says "reuse existing infrastructure" without naming which; the codebase has one
  candidate table. Codebase evidence closes that sub-question (`OQ-002`, Verified) even
  though the PRD never specified it.

## Historical example (Glue, pre-convention)

`apps/quest-ic/glue/docs/specs/incentives/qims-api-contract.md` — the PRD listed budget
management as a Must and stated the vendor system is the record for budgets; the live
OpenAPI had no budget write endpoints, and a codebase migration proved the underlying
table was dropped. Precedence (codebase + live API over PRD) made this a confirmed gap,
not an assumption; the PRD was flagged stale. **Historical — predates the `OQ-NNN`
zero-padding convention** (its open questions are numbered `OQ-1`/`OQ-2` unpadded); do
not pattern-match its ID format.

## See Also

- `../reference/example-initiative/` — conformant synthetic example applying this rule
- `../templates/open-question.md` — open-question format
- `../templates/gap-report.md` — gap-report format
