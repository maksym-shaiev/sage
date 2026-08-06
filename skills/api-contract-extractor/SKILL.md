---
name: api-contract-extractor
description: Produce a verified or proposed API contract spec — from a live OpenAPI/Swagger source, a service codebase, or (Design Mode) a PRD alone when the endpoints an initiative needs do not exist yet. Use for both vendor integrations and net-new feature work. Delegates heavy parsing to a read-isolation subagent.
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

Produce an API contract spec covering both what already exists and what an initiative
needs to build. Runs in one of two modes, per endpoint:

- **Extract Mode** — the endpoint already exists (`existing`/`extension`): verify it
  against a live OpenAPI/Swagger source or the codebase.
- **Design Mode** — the endpoint does not exist yet (`new`): propose its shape from the
  PRD and established codebase patterns; there is nothing to verify against.

Most real initiatives need both modes in the same contract spec — a majority of feature
work is not a pure vendor integration where every endpoint already exists. See
`## Endpoint status tagging` below for the full `status` taxonomy both modes emit into.

## When to use me

- Integrating with an API where a live OpenAPI/Swagger endpoint exists, or the service
  source is available (Extract Mode applies to some or all endpoints).
- Scoping a feature that needs new or extended endpoints that do not exist yet — even
  with no live spec or codebase to extract from at all (Design Mode applies).
- In practice, almost every initiative uses a mix of both modes across its endpoints.

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

**No separate `owner_lane` field on the endpoint.** Ownership already lives one hop
away, on the scope item that implements the endpoint (`scope-mapper`'s `**Lanes:**`
field) — adding a second, parallel ownership tag here would duplicate that with no
consumer to keep the two in sync. If a future deterministic check needs to verify every
`new`/`extension` endpoint has an owning scope item, add the field then, wired to that
check — not speculatively now.

## How to use me

### Step 0 — Triage every endpoint into a mode

For each endpoint the PRD implies, decide before anything else: does it already exist
(→ Extract Mode) or must it be built as part of this initiative (→ Design Mode)? This
triage produces the `status` tag (below) for every row in the final catalog. Most
initiatives produce a mix — do not assume the whole contract is one mode or the other.

### Extract Mode (`existing` / `extension` endpoints)

1. **Locate the live source.** Preference order:
   1. A committed, fetchable OpenAPI/Swagger document (e.g. `{BASE}/openapi.json`),
      ideally the staging/target environment.
   2. A **runtime-generated** docs endpoint, when the service has no static file but
      serves docs on request (e.g. a `/docs` or `/swagger` route rendered from
      annotations at request time) — hit it live rather than assuming a static file
      exists.
   3. The service codebase directly — route/controller definitions, request/response
      DTOs or schemas — when neither of the above exists or does not cover what's
      needed (free-form fields, auth guards, idempotency behaviour OpenAPI can't
      express).
2. **Delegate parsing to a read-isolation subagent** (opencode's built-in `explore`
   agent, or equivalent read-isolation context on other vendors). Large OpenAPI JSON
   will truncate in the main thread; have the subagent parse the saved output and
   return a distilled summary: paths, security schemes, request/response schemas, and
   any free-form objects. This is the efficiency win — do not parse tens of KB inline.
3. **Tag each verified endpoint** `existing` (unchanged) or `extension` (this
   initiative adds fields/behaviour to an existing path) per the precedence rule in
   `../../guides/source-of-truth-precedence.md`.

### Design Mode (`new` endpoints)

There is nothing to verify — this mode produces a **proposal**, not an extraction.

1. **Derive the shape from the PRD** — request/response fields the PRD implies,
   inferred HTTP method and path from the resource it operates on.
2. **Match established patterns**, not invented ones. Look at how the *same* codebase
   (or, for a genuinely new service with no code yet, a sibling service in the same
   ecosystem) shapes comparable endpoints — path structure, envelope shape, error
   format — and follow it. Do not invent a new convention where an established one
   exists.
3. **Tag the endpoint `new`** and mark its shape explicitly proposed (Article III:
   Confirmed-as-proposed, never presented as Verified/already existing — D5).
4. **Anchor it to an owning scope item**, not a live response. Once `scope-mapper` runs,
   its `**Lanes:**` field on the governing scope item records which team/repo will
   build it — this skill does not need its own ownership field for that; see
   `## Endpoint status tagging` above for why.
5. **For a service with no live spec and no code at all** (fully greenfield): tag every
   endpoint `new` and say so explicitly in the contract's Overview section. Do not
   fabricate a contract to extract from, and do not silently treat a Design Mode
   proposal as if it were Extract Mode output.

### Both modes — finishing steps

6. **Build the endpoint catalog** grouped by resource, with method, path, `status`, auth
   scope, request body schema, response schemas (per status code), and key query params.
7. **Document the auth/token model** — schemes, scopes, how tokens are issued/stored.
8. **Inventory free-form fields** (`additionalProperties: true`, no declared keys). List
   expected keys from the PRD but mark them UNVERIFIED pending the owning team.
9. **Detect open questions** — missing write endpoints, read-only configs, dropped
   capabilities (cross-check codebase migrations). Record each via the open-question
   template with a risk level.
10. **Emit the contract spec**: overview (state which mode(s) were used), auth, base
    URL, endpoint catalog (with `status` on every row), response schemas, free-form
    field inventory, state enums, open contract questions, see-also. Cross-link ADRs
    where decisions resolve open questions.

## Output

A `*-api-contract.md` spec: overview, auth, base URL, endpoint catalog (every endpoint
tagged with `status`), response schemas, free-form field inventory, state enums, open
contract questions, see-also.

## Reference output

- `../../reference/example-initiative/widget-api-contract.md` — conformant, small,
  demonstrates all four `status` values and a mixed Extract Mode + Design Mode contract
  (its Overview states which mode produced which endpoint, per Step 10 above)

## See Also

- `../../guides/source-of-truth-precedence.md`
- `../../templates/open-question.md`
- `apps/quest-ic/glue/docs/specs/incentives/qims-api-contract.md` — **historical**
  reference output; predates the `status` tagging convention above (it models a
  vendor-integration initiative where every endpoint already existed, so `status` did
  not yet need to exist) — do not pattern-match its structure for a mixed
  new/existing-endpoint initiative
- `gap-analyzer`, `scope-mapper` skills (consume this output)
