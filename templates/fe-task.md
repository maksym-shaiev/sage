# Template: [FE] Jira Task

> Placeholders in `{{...}}`. Remove this header block when instantiating.
> Summary format: `[FE] {{ITEM-ID}} — {{Title}}`
> Labels: `frontend` | Component: `{{component}}` | Parent epic: `{{epic}}`
> Always include a Figma reference. Always link "Blocked by" the paired [BE] task.

## Story

As a {{user type}}, I {{want/need}} {{capability}} so that {{benefit}}.
(Use the PRD user story verbatim where it exists.)

## Technical Tasks

1. Build {{screen/component}} in the {{SPA}}.
2. {{Fields / interactions / states}}.
3. Call `{{glue endpoint}}` on {{trigger}}; handle loading state.
4. Handle states: success, empty, validation error, service-unavailable.
5. {{Navigation / filters / pagination as needed}}.

## Acceptance Criteria

1. {{UI condition matching the mock}}.
2. {{Behaviour tied to the contract field, e.g. button hidden when `can_redeem` false}}.
3. {{Loading / empty / error states present}}.
4. Design matches the Figma mock.

## Contract

```
{{METHOD}} {{glue endpoint}} → {{response shape consumed by FE}}
```

## Figma

- [{{screen name}}]({{figma url with ?node-id}})

## References

- Spec: `{{scope-of-work path}}` ({{ITEM-ID}})
- Source: {{source ticket}}
- Blocked by: {{BE task key}} ([BE] {{ITEM-ID}})
