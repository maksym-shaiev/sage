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

### {{OQ-ID}} — {{short title}} ({{High | Medium | Low}})

**PRD requirement:** {{what the PRD asks for}}

**API / system reality:** {{what the verified source of truth shows}}

**Technical evidence:**
- {{evidence from live OpenAPI / codebase / migration — concrete, cited}}

**Options:**
1. {{option}} — {{pro/con}}
2. {{option}} — {{pro/con}}

**Owner / question for:** {{team}}

**Status / impact:** {{what it blocks; delivery risk}}

## Deferred (Must — tracked separately)

| Item | Reason | Tracked in |
|---|---|---|
| {{item}} | {{why deferred}} | {{epic/spec TBD}} |
