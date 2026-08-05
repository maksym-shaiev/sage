# Template: Jira Progress Comment

> Posted to the source/tracking ticket at each stage boundary. Placeholders in `{{...}}`.
> Remove this header when used.

## {{Stage}} — {{status, e.g. "Complete" / "Progress Update"}}

{{One-paragraph summary of what was produced this stage.}}

{{Optional: list of artifacts created, with paths.}}

| {{column}} | {{column}} |
|---|---|
| {{row}} | {{row}} |

{{Optional: key decisions or approach confirmations, e.g. "Earn approach: Option A (X-driven)."}}

**Open items / blockers:** {{OQ refs and what they block, or "none"}}

**Next step:** {{the next stage}}.

---

### Variants

- **Coverage update:** include the PRD-requirement → scope-item table.
- **Estimate update:** include the per-phase estimate table + risks with H/M/L.
- **Backlog complete:** include the full task key → summary table grouped by phase.
