---
name: jira-backlog-builder
description: Create paired [BE] + [FE] Jira tasks from a scope-of-work spec, with labels, components, parent epic, and BE-blocks-FE links. Use when turning an approved scope of work into a Jira backlog. Enforces one-pair-then-confirm before bulk creation.
license: MIT
metadata:
  audience: developers
  workflow: backlog
  layer: core
---

## Governance

This skill operates under the [SAGE Constitution](../../constitution/SAGE-CONSTITUTION.md).
It runs after the Discovery Gate (Article IV) has passed — it does not re-check the
Decisions & Assumptions Register itself, and relies on `scope-mapper` having already
enforced D7. **D5 (no invented identifiers)** applies to every Jira field this skill
sets — project keys, component names, and labels must come from the scope-of-work
spec's own header or be confirmed by the human, never guessed.

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

For each non-blocked item, determine task type from the scope item's **`**Lanes:**`
field** — not from free-form prose:

| `**Lanes:**` value | Task type |
|---|---|
| `backend, frontend` (or `frontend, backend`) | `[BE]+[FE]` pair, with blocks link |
| `backend` only | `BE-only` — create only `[BE]` |
| `frontend` only | `FE-only` — create only `[FE]` |
| anything else (a third lane name) | **not yet supported** — see Special cases below; do not guess a label or template for it |

Also extract:
- Task summary prefix: `[BE] {ITEM-ID} — {Title}` / `[FE] {ITEM-ID} — {Title}`
- API contract block (from the scope item's `#### API contract` section)
- Figma reference (from the scope item's `**Figma:**` line — present only when
  `frontend` is in Lanes)
- Implementation note (from the scope item's `**Implementation note:**` line — context
  only, never used to determine task type; that is `**Lanes:**`'s job)

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

**`[BE]` task body** (from `../../templates/be-task.md`):

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

**`[FE]` task body** (from `../../templates/fe-task.md`):

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
| ITEM-03 | — | — | — | BLOCKED (OQ-001) |
```

## Special cases

| Case | Handling |
|---|---|
| **BE-only** (`**Lanes:** backend`) | Create only `[BE]`; no FE task; no blocks link |
| **FE-only** (`**Lanes:** frontend`) | Create only `[FE]`; no BE task; no blocks link |
| **Blocked** | Skip task creation; comment on epic |
| **Even Better If** | Create tasks after all Must items; same format |
| **Third lane** (`**Lanes:**` names anything other than `backend`/`frontend`) | This skill's templates (`be-task.md`/`fe-task.md`) only cover those two. Do not invent a label or template for the unrecognised lane (D5) — flag it in the Step 4 report as "lane not supported — create manually" and move on to the next item. |

## Refuse conditions

- Scope-of-work spec not yet approved — ask the human to confirm approval before starting
- Atlassian MCP not available — cannot create tasks; halt
- Project key or epic key not determinable — ask the human before creating any task
- One-pair-then-confirm not passed — do not continue past the first pair without explicit approval

## Reference input — conformant

`../../reference/example-initiative/scope-of-work.md` — its `**Lanes:**` field on each
item is what this skill's Step 1 parses; small enough to trace by hand end-to-end.

## Historical reference (Glue, pre-convention)

QIMS integration (epic QW-616, 39 tasks + 19 blocks links) and Survey Reminders (epic
QW-746, 18 tasks + dependency links) were both run against scope-of-work files that
predate `**Lanes:**` — Survey Reminders' lane-prefixed IDs (`K-1`/`G-1`/`FE-1`) were
detected by pattern, not by an explicit field. Do not pattern-match that detection
method; the `**Lanes:**` field above is now the only supported mechanism.

## See Also

- `skills/scope-mapper/SKILL.md` — produces the input this skill consumes, including
  the `**Lanes:**` field this skill's Step 1 reads
- `../../templates/be-task.md` — `[BE]` task description format
- `../../templates/fe-task.md` — `[FE]` task description format
