# Reference: Example Initiative ("Widget Favorites")

**This is a synthetic, fictional initiative.** It does not describe any real product,
API, or codebase. It exists solely to be a conformant, constitution-compliant example
of what each Discovery skill's output should look like — every rule the skills and the
[SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md) state, this set actually
demonstrates.

It is **not** a replacement for the historical QuestIC runs (QIMS, Survey Reminders,
Activities V2) referenced elsewhere in this kit as real-world scale examples — those
predate the current conventions and are explicitly marked non-conformant where they
diverge. This set is small on purpose, so every rule it demonstrates is visible in one
sitting.

## What it demonstrates

| Constitution requirement | Where |
|---|---|
| Decisions & Assumptions Register (Article III) with all three states present | `discovery-brief.md` |
| An open question that **stays open and blocks a scope item** (D2, D3) rather than being silently defaulted | OQ-001 → `ITEM-03` in `scope-of-work.md` |
| A claim closed by **evidence** (Verified) | OQ-002 |
| A decision closed **inline** with `Decided-by`/`Decided-on`, no ADR needed | OQ-003 |
| A decision graduating to a **full ADR** with `Decided-by`, `Decided-on`, `Triggered-by`, and rejected alternatives named (D4) | OQ-004 → `decisions/ADR-001-dedicated-favorite-entity.md` |
| Contract-level provenance via `status`: `existing`, `extension`, `new`, and `internal` | `widget-api-contract.md` |
| The Discovery Gate passing with **zero** entries left in the raw Assumed state (D7) | `discovery-brief.md` Decisions & Assumptions Register, final state |
| `ITEM-NN` scope-item IDs with an explicit `**Lanes:**` field (not lane-prefixed IDs) | `scope-of-work.md` |
| A scope item's `**Decision:**` field back-linking to its governing ADR, in **both** directions (ADR's See Also links back too) | `ITEM-01`, `ITEM-02` ↔ `decisions/ADR-001-dedicated-favorite-entity.md` |
| Architecture completeness criterion (Article V) — all 8 dimensions, including one section explicitly stating no cross-repo dimension applies rather than omitting it (D8) | `architecture.md` |

## Files

- `discovery-brief.md` — `discover-prd`'s output, human-approved
- `widget-api-contract.md` — `api-contract-extractor`'s output (mixed Extract Mode +
  Design Mode — the initiative is mostly new-endpoint work, not a pure vendor
  integration; see its Overview for which mode produced which endpoint)
- `decisions/ADR-001-dedicated-favorite-entity.md` — `adr-writer`'s output
- `architecture.md` — `architecture-writer`'s output, satisfying Article V
- `scope-of-work.md` — `scope-mapper`'s output, ready for `jira-backlog-builder`
