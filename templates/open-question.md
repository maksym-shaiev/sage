# Template: Open Contract Question

> One block per unresolved question that blocks or risks delivery. Carry technical evidence
> and a risk level. Placeholders in `{{...}}`. Remove this header when used.

`{{OQ-ID}}` is always `OQ-NNN`, zero-padded (`OQ-001`, not `OQ-1`) — the same ID used in
the Discovery Brief's Decisions & Assumptions Register and in `gap-report.md`.

### {{OQ-NNN}} — {{short title}} ({{High | Medium | Low}} risk)

**PRD requirement:** {{what the PRD requires}}

**API / system reality:** {{what the authoritative source shows — live OpenAPI > codebase > PRD}}

**Technical evidence:**
- {{cited evidence: endpoint absent in OpenAPI, migration that dropped a table, free-form schema, etc.}}

**Owner:** {{team or role — same field name as gap-report.md}}

**Options to propose:**
1. {{option}}
2. {{option}}

**Status / impact:** {{what it blocks; HIGH/MEDIUM/LOW delivery risk and why}}

**Resolution (when resolved):** {{inline: decider + date, no ADR needed | ADR link}} —
matches the Decisions & Assumptions Register's Resolution column (SAGE Constitution
Article III); not every resolved OQ needs an ADR.
