---
name: api-contract-extractor
description: Produce a verified API contract spec from a live OpenAPI/Swagger source (preferred) or a service codebase. Use when integrating with an external/internal API and you need an authoritative endpoint catalog, auth model, and open-question list. Delegates heavy parsing to a read-isolation subagent.
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: discovery
  layer: core
---

## Governance

This skill operates under the [SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md).
**D5 (no invented identifiers)** and **D6 (source-of-truth precedence: live API >
codebase > PRD)** govern this skill directly. Every endpoint this skill emits carries a
`status` tag (Article III, contract-level provenance) — `existing`/`extension` only
after being Verified against a live response or the codebase; `new` marked
Confirmed-as-proposed, never presented as already existing.

## What I do

Extract an authoritative API contract from the live OpenAPI spec (or codebase), and emit a
contract spec: endpoint catalog (method, path, auth scope, request/response schemas), auth
and token model, free-form field inventory, and open contract questions.

## When to use me

- Integrating with an API where a live OpenAPI/Swagger endpoint exists (preferred), or the
  service source is available.

## Source-of-truth precedence

Follow `docs/specs/process/source-of-truth-precedence.md`:
live OpenAPI > service codebase > PRD/Confluence. The live contract is what callers
integrate against; the codebase is authoritative only for what OpenAPI cannot express
(free-form object keys, auth guards, idempotency behaviour).

## How to use me

1. **Fetch the live spec** (e.g. `{BASE}/openapi.json`), preferably the staging/target env.
   Note whether docs are enabled per environment.
2. **Delegate parsing to a read-isolation subagent** (the built-in `explore` agent). Large
   OpenAPI JSON will truncate in the main thread; have the subagent parse the saved output
   and return a distilled summary: paths, security schemes, request/response schemas, and
   any free-form objects. This is the efficiency win — do not parse tens of KB inline.
3. **Build the endpoint catalog** grouped by resource, with method, path, auth scope,
   request body schema, response schemas (per status), and key query params.
4. **Document the auth/token model** — schemes, scopes, how tokens are issued/stored.
5. **Inventory free-form fields** (`additionalProperties: true`, no declared keys). List
   expected keys from the PRD but mark them UNVERIFIED pending the owning team.
6. **Detect open questions** — missing write endpoints, read-only configs, dropped
   capabilities (cross-check codebase migrations). Record each via the open-question
   template with a risk level.
7. **Emit the contract spec** in the project format; add open questions; cross-link ADRs
   where decisions resolve them.

## Output

A `*-api-contract.md` spec: overview, auth, base URL, endpoint catalog, response schemas,
free-form field inventory, state enums, open contract questions, see-also.

## See Also

- `docs/specs/process/source-of-truth-precedence.md`
- `docs/specs/process/_templates/open-question.md`
- `docs/specs/incentives/qims-api-contract.md` — reference output
- `gap-analyzer`, `scope-mapper` skills (consume this output)
