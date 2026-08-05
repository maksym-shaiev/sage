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
  question with technical evidence and a risk level (see `_templates/open-question.md`). Do
  not assume the API will gain it; do not silently drop the requirement.
- **Codebase vs PRD** — if the codebase proves a capability was removed (e.g. a migration
  dropping a table), the removal wins over the PRD's assumption. Flag the PRD as stale.
- **Live API vs codebase** — for endpoint shape, the live contract wins. For undocumented
  internals (free-form config objects, idempotency behaviour), the codebase wins. If the
  codebase is in a different repo than the caller, read the service repo, not the caller.
- **Undocumented free-form fields** — when a schema declares an object as free-form
  (`additionalProperties: true`, no declared keys), treat its keys as UNVERIFIED. Propose
  PRD-derived keys but mark them pending confirmation from the owning team.

## Worked example (QIMS reference run)

- The PRD listed budget management as a Must and stated QIMS is the system of record for
  budgets. The live OpenAPI had **no** budget write endpoints, and a codebase migration
  (`bc214338afae`) proved the `budgets` table was dropped. Precedence: codebase + live API
  over PRD → budget write is a confirmed gap (OQ-1), not an assumption. The PRD was flagged
  stale; an interim approach was decided in ADR-003.
- `points_config` was free-form in the OpenAPI with no declared keys → expiration rules
  marked UNVERIFIED (OQ-2), pending the QIMS team.

## See Also

- `docs/specs/process/ai-sdd-workflow-plan.md` — overall workflow
- `docs/specs/process/_templates/open-question.md` — open-question format
- `docs/specs/process/_templates/gap-report.md` — gap-report format
- `docs/specs/incentives/qims-api-contract.md` — reference contract with open questions
