# ai-sdd-kit

Shared, AI-vendor-agnostic Spec-Driven Development (SDD) workflow kit for Maru apps.

The methodology spine is **SDD**; the **Discovery-to-Delivery (D2D)** pipeline is its
discovery phase. This kit holds the **agnostic core** — guides, templates, and all pipeline
skills — that every app consumes. Vendor shells live in each app.

## Progressive enhancement contract

The agnostic core always produces correct results. Vendor-specific tools (opencode
commands/subagents, Claude Code, Copilot) are an **optional efficiency layer** that produces
the **same** results, faster and cheaper. Removing a vendor tool never breaks correctness —
it only costs more tokens/time. The material efficiency win is **read-isolation subagents**
for heavy parsing (OpenAPI, Figma, codebase); other shells are minor ergonomics.

## Contents

```
ai-sdd-kit/
├── guides/
│   ├── source-of-truth-precedence.md   # live API > codebase > PRD
│   └── estimation-heuristics.md        # AI-assist factors, complexity, parallel timeline
├── templates/
│   ├── be-task.md  fe-task.md          # Jira task description contracts
│   ├── scope-item.md                   # scope-of-work item block
│   ├── gap-report.md  open-question.md # gap analysis (+ evidence + H/M/L risk)
│   └── vp-email.md  jira-comment.md    # stakeholder comms
├── skills/                             # ALL pipeline skills (app-agnostic)
│   ├── discover-prd/                   # orchestrate discovery: PRD + API + Figma + gaps → brief + ADR candidates
│   ├── api-contract-extractor/         # live OpenAPI → verified contract
│   ├── gap-analyzer/                   # PRD vs contract → coverage table + open questions
│   ├── adr-writer/                     # produce ADRs for any significant decision (technical, product, integration, config)
│   ├── scope-mapper/                   # requirements + contract + Figma → scope-of-work spec (creates ADRs first)
│   └── jira-backlog-builder/           # scope of work → [BE]+[FE] Jira tasks (one-pair-then-confirm)
└── vendor-tooling-guide.md             # how to generate vendor shells from core skills
```

## Core vs vendor shells

| Layer | Lives | Examples |
|---|---|---|
| **Core** (this kit) | `ai-sdd-kit` | All pipeline skills, guides, templates |
| **Vendor shells** | each app's `.opencode/`, `.claude/`, `.github/` | thin, optional, on-demand efficiency wrappers over skills |

App-specific Jira conventions (project keys, component names, epic keys) are inputs to the
skills at runtime — not hardcoded in the skills themselves.

## How an app consumes the kit

Mount the kit into the app at `.agents/skills/_shared/` via git submodule (preferred) or
symlink. The agent discovers all skills under `_shared/skills/` automatically.

```bash
# git submodule (run from the app repo root)
git submodule add <ai-sdd-kit-remote-url> .agents/skills/_shared
git commit -m "Add ai-sdd-kit shared SDD workflow"

# OR local symlink (dev-only, not committed)
ln -s ../../../../libs/ai-sdd-kit .agents/skills/_shared
```

See `INSTALL.md` for the full handoff (creating the remote repo + wiring the first app).

## The SDD pipeline

```
discover-prd (core)
  → api-contract-extractor (core) ──┐
  → [figma-inventory — planned]     ├─ Discovery Brief + ADR candidates
  → gap-analyzer (core) ────────────┘
      → adr-writer (core) → ADR(s) for flagged candidates
      → scope-mapper (core) → scope-of-work spec (ADRs written first, then scope items)
          → [human approval]
          → jira-backlog-builder (core) → [BE]+[FE] tasks (one-pair-then-confirm)
              → [stakeholder-update — planned] → Jira comment + VP email
```

Planned additions: `figma-inventory`, `stakeholder-update` (see
`apps/quest-ic/glue/docs/specs/process/ai-sdd-workflow-plan.md`).

## Status

All five pipeline skills are implemented and app-agnostic. Extracted from the QuestIC
reference runs (QIMS epic QW-616, Survey Reminders epic QW-746, Activities V2 epic QW-796).

## Reference implementation

`apps/quest-ic/glue/docs/specs/incentives/` — the specs, scope of work, ADRs, and 39 Jira
tasks produced by a reference run of this pipeline.
