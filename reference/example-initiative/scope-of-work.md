# Widget Favorites — Scope of Works
Type: Spec
Ticket: EX-100
Status: Draft
Updated: 2026-08-06
Description: Phased scope of work for Widget Favorites (synthetic reference example) —
  4 items across 2 lanes, one blocked pending an open question.
---

## Phase 1 — MVP

### ITEM-01 — Show favorite status on the widget list

**PRD:** "Favorited widgets are visually distinguished in the widget list."

**Importance:** Must

**Lanes:** backend

**Notes:** Extends the existing widget-get response; no new endpoint. Backend-only —
the frontend work to render the distinction is `ITEM-04`.

**Decision:** `decisions/ADR-001-dedicated-favorite-entity.md` — this item reads
favorite state from the new `Favorite` entity, not `UserPreference`.

#### API contract

GET /api/widgets/{id}
Auth: member session
Request: none (path param only)
Response 200: existing shape + `is_favorited: boolean`

**Implementation note:** extends an existing endpoint (`extension` — see
`widget-api-contract.md`); no migration beyond the `Favorite` table from `ITEM-02`.

---

### ITEM-02 — Favorite / unfavorite a widget

**PRD:** "Member can mark/unmark a widget as favorite from the dashboard."

**Importance:** Must

**Lanes:** backend

**Notes:** Creates the `Favorite` entity and the endpoint that writes to it.

**Decision:** `decisions/ADR-001-dedicated-favorite-entity.md`

#### API contract

POST /api/widgets/{id}/favorite
Auth: member session
Request: { "favorited": boolean }
Response 200: { "id": "{id}", "is_favorited": boolean }
Response 404: widget not found

**Implementation note:** new endpoint (`new` — see `widget-api-contract.md`); includes
the `Favorite` table migration. The favorite-limit check (409 response) is deferred to
`ITEM-03` once OQ-001 resolves — this item ships without a limit.

---

### ITEM-03 — Enforce a per-member favorite limit

**PRD:** "There is a reasonable limit on how many widgets can be favorited."

**Importance:** Even Better If

**Lanes:** backend

**Status: BLOCKED** — awaiting OQ-001 (favorite limit value). No tasks until resolved.

**Notes:** The `Favorite` entity from `ITEM-02` supports an efficient `COUNT` query the
moment a limit value is confirmed (see ADR-001, Consequence 1) — this item is blocked
only on the *value*, not on any architectural unknown.

---

## Phase 2 — Frontend

### ITEM-04 — Favorite toggle UI on the dashboard

**PRD:** "Member can mark/unmark a widget as favorite from the dashboard." /
"Favorited widgets are visually distinguished in the widget list."

**Importance:** Must

**Lanes:** frontend

**Notes:** Depends on `ITEM-01` (for `is_favorited` in the list response) and `ITEM-02`
(for the toggle action). Sequenced after Phase 1 for that reason, not because it is
lower priority.

**Figma:** [Dashboard — Widget Card](https://figma.example/file/EX-100?node-id=1%3A1)

#### API contract

POST /api/widgets/{id}/favorite → { "id": "{id}", "is_favorited": boolean }
GET /api/widgets/{id} → existing shape + is_favorited (consumed for list rendering)

**Implementation note:** pure consumer of `ITEM-01`/`ITEM-02`'s contracts; no new
backend work.

---

## Delivery Estimate

### Complexity per item

| ITEM-ID | Title | BE complexity | BE days | FE complexity | FE days |
|---|---|---|---|---|---|
| ITEM-01 | Show favorite status | Low | 1 | — | — |
| ITEM-02 | Favorite/unfavorite endpoint | Medium | 1.5 | — | — |
| ITEM-03 | Favorite limit (BLOCKED) | — | unestimated | — | — |
| ITEM-04 | Favorite toggle UI | — | — | Low | 1 |

AI-assist factors applied: BE ~50–60% faster; FE ~25–30% faster vs. unassisted.

### Phase timeline

| Phase | Items | BE days | FE days | Phase duration |
|---|---|---|---|---|
| Phase 1 | ITEM-01, ITEM-02 | 2.5 | — | 2.5 |
| Phase 2 | ITEM-04 | — | 1 | 1 |

Phase duration = max(BE days, FE days) — BE and FE run in parallel within a phase; here
Phase 2 has no BE work, so it is FE-only duration.
Total = sum of phase durations + 20% buffer = 3.5 × 1.2 ≈ **4.2 days**.

**Total estimate: ~1 week** (1 BE + 1 FE, AI-assisted, part-time on this initiative).

> ITEM-03 is excluded from the estimate until OQ-001 resolves, per the estimation
> guide's rule that blocked items are not estimated.

## See Also

- `discovery-brief.md` — the approved Discovery Brief this scope was mapped from
- `widget-api-contract.md` — the contract referenced in every item above
- `architecture.md` — the architecture spec this scope implements
- `decisions/ADR-001-dedicated-favorite-entity.md` — governs ITEM-01, ITEM-02
