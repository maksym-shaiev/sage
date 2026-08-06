# Template: Gap Report

> Output of the gap-analyzer skill. PRD requirements vs verified contract.
> Placeholders in `{{...}}`. Remove this header when used.

## Coverage Summary

| PRD requirement | Importance | Covered by | Status |
|---|---|---|---|
| {{requirement}} | {{Must/EBI}} | {{ITEM-ID / deferred / blocked}} | {{✅ / ⚠️ / ❌}} |

Rule: every **Must** requirement must map to a scope item, a documented deferral, or a
blocked status. Nothing is silently dropped.

## Gaps / Open Questions

`{{OQ-ID}}` is always `OQ-NNN`, zero-padded (`OQ-001`, not `OQ-1`), sequential within
the initiative — matches the ID used in the Discovery Brief's Decisions & Assumptions
Register (SAGE Constitution Article III).

### {{OQ-NNN}} — {{short title}} ({{High | Medium | Low}})

**PRD requirement:** {{what the PRD asks for}}

**API / system reality:** {{what the verified source of truth shows}}

**Technical evidence:**
- {{evidence from live OpenAPI / codebase / migration — concrete, cited}}

**Options:**
1. {{option}} — {{pro/con}}
2. {{option}} — {{pro/con}}

**Owner:** {{team or role}}

**ADR required:** {{Yes — Subtype | No}} — feeds the Discovery Brief's `ADR required`
column (see `discover-prd` Step 4b)

**Status / impact:** {{what it blocks; delivery risk}}

## Deferred (Must — tracked separately)

| Item | Reason | Tracked in |
|---|---|---|
| {{item}} | {{why deferred}} | {{epic/spec TBD}} |
