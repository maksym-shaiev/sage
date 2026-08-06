# Template: [BE] Jira Task

> Placeholders in `{{...}}`. Remove this header block when instantiating.
> Summary format: `[BE] {{ITEM-ID}} — {{Title}}`
> Labels: `backend` | Component: `{{component}}` | Parent epic: `{{epic}}`

## Story

As a {{user type}}, I {{want/need}} {{capability}} so that {{benefit}}.
(Use the PRD user story verbatim where it exists.)

## Technical Tasks

1. {{For endpoints matching an established pass-through/proxy pattern in this
   codebase}}: wire the route to {{upstream method+path}} using the existing
   generic pattern — **no custom handler class** unless justified below.
2. {{For exceptions only}}: implement a custom handler/processor — **justified
   exception** because {{reason the standard pattern cannot express it}}. Reference
   the architecture spec's integration section.
3. {{Entity/migration/config steps as needed}}.
4. Secure the operation per the codebase's established auth pattern ({{role/scope
   required}}).
5. Write unit tests for {{class/module}}.

{{Adapt step 1/2 wording to whatever this codebase's actual established pattern is —
do not invent framework-specific syntax; use what is visible in the codebase (D5).}}

## Acceptance Criteria

1. {{Verifiable condition tied to the contract}}.
2. {{Error/edge handling condition}}.
3. {{Idempotency / security condition where relevant}}.

## Contract

```
{{METHOD}} {{glue endpoint}}
Auth: {{ROLE}}
Request: {{shape}}
Response {{code}}: {{shape}}
```

Upstream calls:
- {{upstream method+path}} {{notes: token scope, idempotency}}

## References

- Spec: `{{scope-of-work path}}` ({{ITEM-ID}})
- Architecture: `{{architecture path}}`
- Contract: `{{api-contract path}}`
- Decision: {{ADR link if applicable}}
- Source: {{source ticket}}
