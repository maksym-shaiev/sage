---
name: jira-backlog-builder
description: Create paired [BE] + [FE] Jira tasks from a scope-of-work spec, with labels, components, parent epic, and BE-blocks-FE links. Use when turning an approved scope of work into a Jira backlog. Enforces one-pair-then-confirm before bulk creation.
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: backlog
  layer: core
---

## What I do

Turn an approved scope-of-work spec into Jira tasks. Creates paired `[BE]` + `[FE]` tasks
per scope item, links BE → blocks → FE, and enforces a one-pair-then-confirm gate before
bulk creation. Uses the Atlassian MCP tools throughout.

## When to use me

- Scope-of-work spec is approved by a human
- Ready to create the Jira backlog for a delivery phase
- Do NOT run on a scope-of-work spec that still has unresolved Blocked items — skip those

## Pre-conditions

- Approved scope-of-work spec (path)
- Jira project key — read from the spec `Ticket:` metadata field (e.g. `QW-987`); extract
  the project prefix (e.g. `QW`). Ask the human if ambiguous.
- Parent epic key — read from the spec overview section. Ask the human if missing.
- Atlassian MCP server available and authenticated

## Inputs

| Input | Source |
|---|---|
| Scope-of-work spec path | Human provides or `scope-mapper` output |
| Project key | Spec `Ticket:` header prefix |
| Epic key | Spec overview (e.g. "Parent epic: QW-616") |
| Component name(s) | Optional — ask if not in spec |
| Label overrides | Optional — defaults: `backend` / `frontend` |

## Steps

### Step 1 — Read and parse the scope-of-work spec

Extract all items in order:
- Must phases first (Phase 1, Phase 2, Phase 3)
- Even Better If items after all Must phases
- Note Blocked items but **do not create tasks for them** (see Blocked items below)

For each non-blocked item, determine:
- Task type: `[BE]+[FE]` pair / `BE-only` / `FE-only`
- Task summary prefix: `[BE] {ITEM-ID} — {Title}` / `[FE] {ITEM-ID} — {Title}`
- API contract block (from the scope item's `#### API contract` section)
- Figma reference (from the scope item's `**Figma:**` line — omit for BE-only)
- Implementation note (from the scope item's `**Implementation note:**` line)

### Step 2 — One-pair-then-confirm (ENFORCED)

1. Create the **first `[BE]` task** for the first non-blocked item
2. Create the **first `[FE]` task** for the same item (skip if BE-only; create only `[FE]`
   if FE-only)
3. Create the **blocks link**: BE task → blocks → FE task (skip if BE-only or FE-only)
4. **STOP** — present the created pair (or single task) to the human:
   - Show: task keys, summaries, link created
   - Ask: "Approve to continue with remaining {n} items?"
5. **Do not create any further tasks** until the human explicitly approves continuing.

Once approved: create remaining items one-by-one in spec order without further stops,
unless the human asks to pause.

### Step 3 — Create tasks

For each non-blocked item, using `createJiraIssue` and `createIssueLink`:

**`[BE]` task body** (from `_shared/templates/be-task.md`):

```
## Story
As a {user type}, I {want/need} {capability} so that {benefit}.
(Use the PRD user story from the scope item verbatim where it exists.)

## Technical Tasks
1. {Derived from the scope item's implementation note and the app's established patterns.
   Do not invent framework-specific details — use what is visible in the codebase.}
2. {Additional steps as needed.}
3. Write unit tests for the changed/new code.

## Acceptance Criteria
1. {Verifiable condition tied to the API contract.}
2. {Error / edge case handling condition.}
3. {Auth / idempotency condition where applicable.}

## Contract
{METHOD} {/endpoint}
Auth: {scheme}
Request: {shape}
Response {code}: {shape}

## References
- Spec: {scope-of-work path} ({ITEM-ID})
- Decision: {ADR link if present in scope item — omit if none}
```

Fields: `summary = "[BE] {ITEM-ID} — {Title}"`, `issueTypeName = "Task"`,
`labels = ["backend"]`, `parent = {epic key}`.

**`[FE]` task body** (from `_shared/templates/fe-task.md`):

```
## Story
As a {user type}, I {want/need} {capability} so that {benefit}.

## Technical Tasks
1. Build {screen/component} per the Figma reference.
2. {Fields / interactions / states from the scope item.}
3. Call `{/endpoint}` on {trigger}; handle loading, empty, error states.
4. Match design to Figma mock.

## Acceptance Criteria
1. {UI condition matching the Figma mock.}
2. {Behaviour tied to an API contract field.}
3. Loading / empty / error states present.
4. Design matches Figma.

## Contract
{METHOD} {/endpoint} → {response shape consumed by FE}

## Figma
{screen link from scope item}

## References
- Spec: {scope-of-work path} ({ITEM-ID})
- Blocked by: {[BE] task key}
```

Fields: `summary = "[FE] {ITEM-ID} — {Title}"`, `issueTypeName = "Task"`,
`labels = ["frontend"]`, `parent = {epic key}`.

**Blocks link:** use `createIssueLink` with `type = "Blocks"`,
`inwardIssue = {BE key}`, `outwardIssue = {FE key}`.

### Blocked items

For each Blocked item in the scope-of-work spec:
- Do NOT create `[BE]` or `[FE]` tasks
- Add a comment to the parent epic noting: item ID, title, blocking OQ-ID, and that tasks
  will be created once the OQ resolves

### Step 4 — Report

After all tasks are created, output a summary table:

```
| ITEM-ID | [BE] key | [FE] key | Blocks link | Notes |
|---|---|---|---|---|
| ITEM-01 | QW-1001 | QW-1002 | ✓ | |
| ITEM-02 | QW-1003 | — | — | BE-only |
| ITEM-03 | — | — | — | BLOCKED (OQ-1) |
```

## Special cases

| Case | Handling |
|---|---|
| **BE-only** (scope item says "No FE task") | Create only `[BE]`; no FE task; no blocks link |
| **FE-only** (scope item says "No BE task") | Create only `[FE]`; no BE task; no blocks link |
| **Blocked** | Skip task creation; comment on epic |
| **Even Better If** | Create tasks after all Must items; same format |

## Refuse conditions

- Scope-of-work spec not yet approved — ask the human to confirm approval before starting
- Atlassian MCP not available — cannot create tasks; halt
- Project key or epic key not determinable — ask the human before creating any task
- One-pair-then-confirm not passed — do not continue past the first pair without explicit approval

## Reference outputs

- 39 tasks + 19 blocks links — QIMS integration (epic QW-616)
- 18 tasks + dependency links — Survey Reminders (epic QW-746)

## See Also

- `skills/scope-mapper/SKILL.md` — produces the input this skill consumes
- `templates/be-task.md` — `[BE]` task description format
- `templates/fe-task.md` — `[FE]` task description format
