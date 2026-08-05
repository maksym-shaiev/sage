# Template: Scope-of-Work Item

> One block per PRD requirement. Placeholders in `{{...}}`. Remove this header when used.
> Every item maps to a [BE] and/or [FE] task. Mark FE-only / BE-only / blocked explicitly.

### {{ITEM-ID}} — {{Title}}

**PRD:** *"{{PRD user story verbatim}}"*

**Importance:** {{Must | Even Better If}}

**Notes:** {{PRD notes verbatim + any clarifying context}}

{{If FE-only}}: **No BE task** — {{reason, e.g. covered by ITEM-XX}}.
{{If BE-only}}: **No FE task** — {{reason, e.g. pure backend plumbing}}.
{{If blocked}}: **Status: BLOCKED** — awaiting {{OQ-ref}}. No Jira tasks until resolved.

**Decision:** {{ADR link if a decision governs this item}}

**Figma:** [{{screen}}]({{figma url}})  ({{or N/A for behavioural/BE-only items}})

#### Glue contract

```
{{METHOD}} {{glue endpoint}}
Auth: {{ROLE}}
Request: {{shape}}
Response {{code}}: {{shape}}
```

**Upstream calls:** {{list, with token scope + idempotency notes}}

**State implementation:** {{generic `source.*.provider.read/write` — no custom class | Custom `{{Class}}` (reason)}}
