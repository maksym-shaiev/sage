# Use a Dedicated Favorite Entity, Not the Generic Preference Table
Type: Decision
Subtype: Architecture
Ticket: EX-100
Status: Accepted
Updated: 2026-08-06
Description: Why favorite state is stored in a new dedicated entity rather than reused
  inside the existing generic UserPreference table, for the Widget Favorites initiative.
Triggered-by: OQ-004
Decided-by: Jordan Lee, Tech Lead
Decided-on: 2026-08-06
---

## Context

The Widget Favorites initiative needs to persist, per member, which widgets are
favorited. The codebase already has a generic `UserPreference` key-value table used for
several unrelated per-user settings (`[Verified: src/models/UserPreference, 2026-08-05]`).
The PRD does not specify storage — favoriting is described only as a user-facing
capability (`[Verified: PRD §3.2, "Favorite widgets"]`).

| Option | Approach |
|---|---|
| A — Dedicated `Favorite` entity | New table: `member_id`, `widget_id`, `created_at` |
| B — Reuse `UserPreference` | Store `favorite:{widget_id} = true` as a preference key |

## Problem

Favorite state must be queryable efficiently (list all favorites for a member; check
favorite status for a widget in a list response) and must support the limit rule once
OQ-001 resolves. A decision was needed before `ITEM-01`/`ITEM-02` could be scoped,
because the storage choice determines both endpoints' implementation.

## Options

### Option A — Dedicated `Favorite` entity

| | |
|---|---|
| **Pro** | Indexable on `(member_id, widget_id)`; a `COUNT` query directly supports the OQ-001 limit check once it resolves |
| **Pro** | Favorite is a first-class relationship, not an overloaded string key — clearer for future features (e.g. "recently favorited") |
| **Con** | One new table + migration |

### Option B — Reuse `UserPreference`

| | |
|---|---|
| **Pro** | No new table |
| **Con** | Counting a member's favorites means scanning and pattern-matching preference keys — no index supports it |
| **Con** | Conflates a stable relationship with a loosely-typed settings bag |

## Decision

**Option A — dedicated `Favorite` entity.** Rejected Option B because the limit check
in OQ-001 (still open, blocking `ITEM-03`) will require an efficient count query the
moment it resolves, and `UserPreference`'s key-value shape cannot support that without
a full scan. A new table with a proper index is one migration now versus a forced
re-migration later.

> *"We already know a limit is coming — don't paint ourselves into the preference-table
> corner just to save one migration."*
> — Jordan Lee, Tech Lead, 2026-08-06

## Consequences

1. **Positive outcomes** — `ITEM-02` can implement the limit check (once OQ-001
   resolves) as a single indexed `COUNT` query; no re-migration needed later.
2. **Risks and limitations** — one additional table and migration to review and ship
   before `ITEM-01`/`ITEM-02` can merge.
3. **PRD/mock adjustments required** — none; the PRD does not specify storage, so no
   requirement or mock changes.
4. **Downstream obligations** — `ITEM-01` and `ITEM-02` must use the new `Favorite`
   entity, not `UserPreference`. Whoever resolves OQ-001 (the limit value) will query
   this table directly.
5. **Deferral notes** — none; this decision is not blocked on anything.

## See Also

- Triggered by OQ-004 in `../discovery-brief.md`
- Governs `ITEM-01` and `ITEM-02` in `../scope-of-work.md` (back-linked via each item's
  `**Decision:**` field)
- `../widget-api-contract.md` — the `widget.favorite` endpoint this entity backs
