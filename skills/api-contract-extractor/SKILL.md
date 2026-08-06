---
name: api-contract-extractor
description: Produce a verified API contract spec from a live OpenAPI/Swagger source (preferred) or a service codebase. Use when integrating with an external/internal API and you need an authoritative endpoint catalog, auth model, and open-question list. Delegates heavy parsing to a read-isolation subagent.
license: MIT
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

Follow `../../guides/source-of-truth-precedence.md`:
live OpenAPI > service codebase > PRD/Confluence. The live contract is what callers
integrate against; the codebase is authoritative only for what OpenAPI cannot express
(free-form object keys, auth guards, idempotency behaviour).

## Endpoint status tagging

Most feature initiatives are **not** pure vendor integrations against a fully-existing
API — many endpoints an initiative needs do not exist yet and must be designed as part
of Discovery. Tag every endpoint this skill emits with a `status`:

| `status` | Meaning | Provenance (Article III) |
|---|---|---|
| `existing` | Already live, unchanged | Verified against the live response/OpenAPI entry |
| `extension` | Base path exists; this initiative adds fields/behaviour to it | Base path Verified; the addition itself is Confirmed-as-proposed until built |
| `new` | Does not exist yet; must be built as part of this initiative | Confirmed-as-proposed only — never presented as already existing (D5) |
| `internal` | This repo/service only; no upstream dependency to verify against | N/A — no external anchor applies |

For `existing`/`extension` endpoints, verify against the live spec or codebase per the
precedence above before tagging. For `new` endpoints, there is nothing to verify against
— propose the shape from the PRD, mark it explicitly proposed, and anchor it to the
scope item that owns building it (`scope-mapper` output) rather than to a live response.
See `../../reference/example-initiative/widget-api-contract.md` for a worked example
covering all four values.

## How to use me

1. **Determine what's verifiable.** For each endpoint the PRD implies, check whether it
   already exists (live spec / codebase) or must be newly built. This determines its
   `status` (above) before anything else.
2. **Fetch the live spec** (e.g. `{BASE}/openapi.json`), preferably the staging/target
   env, for the `existing`/`extension` portion. Note whether docs are enabled per
   environment. If no live spec exists at all (a greenfield service), fall back to the
   codebase, or — for a genuinely new service with no code yet — proceed with every
   endpoint tagged `new` and say so explicitly; do not fabricate a contract to extract.
3. **Delegate parsing to a read-isolation subagent** (opencode's built-in `explore`
   agent, or equivalent read-isolation context on other vendors). Large OpenAPI JSON
   will truncate in the main thread; have the subagent parse the saved output
   and return a distilled summary: paths, security schemes, request/response schemas, and
   any free-form objects. This is the efficiency win — do not parse tens of KB inline.
4. **Build the endpoint catalog** grouped by resource, with method, path, `status`, auth
   scope, request body schema, response schemas (per status code), and key query params.
5. **Document the auth/token model** — schemes, scopes, how tokens are issued/stored.
6. **Inventory free-form fields** (`additionalProperties: true`, no declared keys). List
   expected keys from the PRD but mark them UNVERIFIED pending the owning team.
7. **Detect open questions** — missing write endpoints, read-only configs, dropped
   capabilities (cross-check codebase migrations). Record each via the open-question
   template with a risk level.
8. **Emit the contract spec**: overview, auth, base URL, endpoint catalog (with `status`
   on every row), response schemas, free-form field inventory, state enums, open contract
   questions, see-also. Add open questions; cross-link ADRs where decisions resolve them.

## Output

A `*-api-contract.md` spec: overview, auth, base URL, endpoint catalog (every endpoint
tagged with `status`), response schemas, free-form field inventory, state enums, open
contract questions, see-also.

## Reference output

- `../../reference/example-initiative/widget-api-contract.md` — conformant, small,
  demonstrates all four `status` values

## See Also

- `../../guides/source-of-truth-precedence.md`
- `../../templates/open-question.md`
- `apps/quest-ic/glue/docs/specs/incentives/qims-api-contract.md` — **historical**
  reference output; predates the `status` tagging convention above (it models a
  vendor-integration initiative where every endpoint already existed, so `status` did
  not yet need to exist) — do not pattern-match its structure for a mixed
  new/existing-endpoint initiative
- `gap-analyzer`, `scope-mapper` skills (consume this output)
