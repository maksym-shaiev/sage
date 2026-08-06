# Widget Favorites — API Contract
Type: Integration
Ticket: EX-100
Status: Draft
Updated: 2026-08-06
Description: Endpoint catalog for the Widget Favorites initiative (synthetic reference
  example), covering one extended endpoint, one new endpoint, one internal-only
  endpoint, and the one pre-existing endpoint it builds on.
---

## Overview

`Widget Favorites` lets a member mark widgets on their dashboard as favorites. It
extends one existing endpoint, adds one new endpoint, and adds one internal-only
endpoint with no upstream dependency.

## Authentication

All member-facing endpoints require a member session bearer token. The internal
endpoint is service-to-service only, authenticated by a shared internal token.

## Endpoints

| ID | Method | Path | `status` | Purpose |
|---|---|---|---|---|
| `widgets.list` | `GET` | `/api/widgets` | `existing` | Already returns the member's dashboard widgets. `[Verified: live OpenAPI, 2026-08-05]` |
| `widget.get` | `GET` | `/api/widgets/{id}` | `extension` | Base path Verified against live OpenAPI; this initiative adds a new `is_favorited` boolean field. `[Confirmed: addition scoped in ITEM-01]` |
| `widget.favorite` | `POST` | `/api/widgets/{id}/favorite` | `new` | Does not exist yet. `[Confirmed: proposed shape below, owned by ITEM-02 — not a live response]` |
| `widget.favorites-cache-warm` | `POST` | `/api/internal/favorites-cache-warm` | `internal` | Internal cache-warming hook; no external caller, no upstream dependency to verify against. |

### `widget.get` (extension) — added field

```
GET /api/widgets/{id}
Response 200 (existing fields unchanged, plus):
{
  "is_favorited": boolean   // new field, this initiative
}
```

### `widget.favorite` (new) — proposed shape

```
POST /api/widgets/{id}/favorite
Auth: member session
Request: { "favorited": boolean }
Response 200: { "id": "{id}", "is_favorited": boolean }
Response 404: widget not found
Response 409: favorite limit reached — see OQ-001, ITEM-03
```

This shape is **Confirmed-as-proposed** (Article III), not Verified — it does not exist
in the live API. It is anchored by `ITEM-02` in `scope-of-work.md`, not by a live
response.

## Free-form fields

None declared on these endpoints.

## Open contract questions

See `discovery-brief.md`'s Decisions & Assumptions Register — OQ-001 (favorite limit,
open and blocking `ITEM-03`) and OQ-002 (which entity stores favorite state, resolved
by codebase evidence) both originate from this contract.

## See Also

- `../../guides/source-of-truth-precedence.md` — precedence rule applied above
- `discovery-brief.md` — Decisions & Assumptions Register referencing this contract
- `scope-of-work.md` — scope items implementing `widget.get` (extension) and
  `widget.favorite` (new)
