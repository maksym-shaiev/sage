# Widget Favorites — Discovery Brief
Type: Spec
Ticket: EX-100
Status: Approved
Updated: 2026-08-06
Description: Discovery Brief for the Widget Favorites initiative (synthetic reference
  example) — requirement inventory, verified contract, gap analysis, and the Decisions
  & Assumptions Register at the point of human approval.
---

## PRD Summary

Members want to mark widgets on their dashboard as favorites for faster access. Product
goal: reduce time-to-find for frequently used widgets. Stakeholder: Dashboard team PO.

## Requirement Inventory

| ID | Requirement | Importance | Figma screen |
|---|---|---|---|
| REQ-01 | Member can mark/unmark a widget as favorite from the dashboard | Must | Dashboard — Widget Card |
| REQ-02 | Favorited widgets are visually distinguished in the widget list | Must | Dashboard — Widget Card |
| REQ-03 | There is a reasonable limit on how many widgets can be favorited | Even Better If | N/A — no mock exists yet |

## Verified API Contract

Reference: `widget-api-contract.md`
Key findings: one existing endpoint (`GET /api/widgets`) already returns the dashboard
widget list; one endpoint needs a new field (`is_favorited` on `GET /api/widgets/{id}`
— `extension`); one endpoint does not exist yet (`POST /api/widgets/{id}/favorite` —
`new`); no free-form fields on any of them.

## Figma Screen Inventory

| Screen | Figma node | PRD requirements covered |
|---|---|---|
| Dashboard — Widget Card | `EX-100:1` | REQ-01, REQ-02 |

REQ-03 has no design reference — no limit-reached UI exists yet; see OQ-001.

## Gap Report

Reference: inline below (short enough not to warrant a separate file)

| PRD requirement | Importance | Covered by | Status |
|---|---|---|---|
| REQ-01 | Must | ITEM-01, ITEM-02, ITEM-04 | ✅ covered |
| REQ-02 | Must | ITEM-01, ITEM-04 | ✅ covered |
| REQ-03 | Even Better If | ITEM-03 | ⚠️ blocked — OQ-001 |

## Open Questions

| OQ-ID | Question | Risk | Owner | ADR required |
|---|---|---|---|---|
| OQ-001 | What is the favorite limit per member, if any? | Medium | Dashboard PO | No — a config value once decided, not an architecture choice |
| OQ-002 | Which existing entity, if any, already models per-member widget relationships we could extend? | Low | — | No |
| OQ-003 | Should favorite counts be visible to other members, or owner-only? | Low | Dashboard PO | No — single-option product call once asked |
| OQ-004 | Should favorite state live in a new dedicated entity or the existing generic preference table? | High | Tech Lead | Yes — Architecture |

## Decisions & Assumptions Register

Per SAGE Constitution Article III. State at the time this brief was presented for
approval — every row below is Verified, Confirmed, or explicitly Open-and-blocking; the
gate (Article IV) would not have passed otherwise (D7):

| ID | Statement | State | Evidence / Decided-by (+date) | Blocks | Resolution |
|---|---|---|---|---|---|
| OQ-001 | Favorite limit per member is undetermined — no PRD value, no precedent found | Open — not Assumed | — | ITEM-03 | pending; `ITEM-03` scoped as Blocked, not silently included |
| OQ-002 | No existing entity models widget-favorite relationships; `UserPreference` is the closest generic candidate | Verified | `[Verified: src/models/UserPreference, 2026-08-05]` | — | resolved by codebase evidence, feeds OQ-004 |
| OQ-003 | Favorite counts are owner-only, not publicly visible | Confirmed | Dashboard PO — 2026-08-06 | — | inline (no ADR needed — single-option call once asked, per `adr-writer`'s "do not create an ADR for" list) |
| OQ-004 | Favorite state stored in a new dedicated `Favorite` entity, not `UserPreference` | Confirmed | Jordan Lee, Tech Lead — 2026-08-06 | — | `decisions/ADR-001-dedicated-favorite-entity.md` |

**Note on OQ-001:** this is the demonstration case for D2/D3 — the agent did not choose
a plausible default limit (e.g. "10, seems reasonable") and proceed. It is recorded as
genuinely unresolved and its dependent item (`ITEM-03`) is explicitly Blocked in
`scope-of-work.md`, not scoped as if the question were settled.

## Handoff note

Ready for scope-mapper — every register entry above is Verified, Confirmed, or
Open-and-blocking (D7). Approved by: Dashboard team PO, 2026-08-06, against this
version of this brief.

## See Also

- `widget-api-contract.md` — the verified/proposed contract referenced above
- `decisions/ADR-001-dedicated-favorite-entity.md` — resolves OQ-004
- `architecture.md` — the architecture spec built on this brief and its ADR
- `scope-of-work.md` — the scope items this brief hands off to
