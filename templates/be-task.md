# Template: [BE] Jira Task

> Placeholders in `{{...}}`. Remove this header block when instantiating.
> Summary format: `[BE] {{ITEM-ID}} — {{Title}}`
> Labels: `backend` | Component: `{{component}}` | Parent epic: `{{epic}}`

## Story

As a {{user type}}, I {{want/need}} {{capability}} so that {{benefit}}.
(Use the PRD user story verbatim where it exists.)

## Technical Tasks

1. {{For generic-chain endpoints}}: Add `shortName: '{{short_name}}'` to `{{Client}}::mapUri()` → `{{upstream method+path}}`; declare `provider: 'source.{{src}}.provider.read'` (or `processor: 'source.{{src}}.provider.write'`) on the `#[ApiResource]` operation — **no custom State class**.
2. {{For exceptions only}}: Implement `{{CustomProcessorOrProvider}}` — **custom State class, justified exception** because {{reason the generic chain cannot express it}}. Reference the architecture spec's client-service section.
3. {{Entity/migration/config steps as needed}}.
4. Secure the operation with `is_granted('{{ROLE}}')`.
5. Write unit tests for {{class}}.

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
