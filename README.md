# SAGE — Spec-as-Governance Engine

Shared, AI-vendor-agnostic **Discovery-phase** workflow kit for Maru apps.

SAGE is a two-phase methodology: **Discovery** (human-approved architecture, ADRs, and
scope of work) and **Delivery** (AI-generated tickets, code, and tests, gated by a
deterministic CI audit). This repository ships the **Discovery phase and its governance
conventions only** — guides, templates, and all Discovery-pipeline skills, consumed
identically by every app.

**Delivery-phase enforcement (the `spec:*` block schema, the CI audit, test generation)
is implemented per app, behind an adapter contract, and is not shipped here.** Glue
(`apps/quest-ic/glue`) is the reference implementation of that adapter — see
`docs/specs/testing/architecture.md` there for its concrete 9-check audit toolchain.

## Governance

`constitution/SAGE-CONSTITUTION.md` is the single, versioned source of governance for
every repo that mounts this kit. It has two parts:

- **Article I — Universal Governance**, extracted from rules Glue and Kato had each
  independently written and had already begun to drift from each other (see
  `CHANGELOG.md` for the specific drift this fixed). A consuming repo's `AGENTS.md`
  states the rule headlines and points here for full text, instead of duplicating it.
- **Article II — Discovery Governance**, new rules (D1-D8) governing how AI agents may
  write Discovery artifacts: no unconfirmed claim may be written as settled fact, every
  gap becomes a recorded open question rather than a silent default, and the
  human-approval gate mechanically requires that record to be empty of unresolved
  assumptions before it can pass.

All six skills below load this constitution and reference the specific articles they
implement.

## Progressive enhancement contract

The agnostic core always produces correct results. Vendor-specific tools (opencode
commands/subagents, Claude Code, Copilot) are an **optional efficiency layer** that produces
the **same** results, faster and cheaper. Removing a vendor tool never breaks correctness —
it only costs more tokens/time. The material efficiency win is **read-isolation subagents**
for heavy parsing (OpenAPI, Figma, codebase); other shells are minor ergonomics.

## Contents

```
sage/
├── CHANGELOG.md                        # adoption history, provenance for the constitution
├── constitution/
│   └── SAGE-CONSTITUTION.md            # governance: universal rules (G1-G5), Discovery
│                                        # rules (D1-D8), provenance model, the gate,
│                                        # architecture completeness, extension slots —
│                                        # app-agnostic; no repo or tool named anywhere
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
| **Core** (this kit) | `sage` | All Discovery-pipeline skills, guides, templates, the constitution |
| **Vendor shells** | each app's `.opencode/`, `.claude/`, `.github/` | thin, optional, on-demand efficiency wrappers over skills |

App-specific Jira conventions (project keys, component names, epic keys) are inputs to the
skills at runtime — not hardcoded in the skills themselves.

## How an app consumes the kit

Mount the kit into the app at `.agents/skills/_shared/` as a **pinned git submodule** —
this is the standard, committable way every app consumes SAGE. The agent discovers all
skills under `_shared/skills/` automatically.

```bash
# git submodule (run from the app repo root), pinned to a released tag
git submodule add git@github.com:maksym-shaiev/sage.git .agents/skills/_shared
cd .agents/skills/_shared && git checkout <tag> && cd -   # no tag cut yet — see Status
git add .gitmodules .agents/skills/_shared
git commit -m "Add SAGE (Discovery phase) as a pinned submodule"
```

A local symlink (`ln -s <path-to>/libs/sage .agents/skills/_shared`) is acceptable for
solo, uncommitted dev-loop iteration only — it cannot be shared with another developer
or committed, and both reference apps (Glue, Kato) use the submodule form.

See `INSTALL.md` for the full handoff.

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

Planned additions: `figma-inventory`, `stakeholder-update`.

## Status

All six pipeline skills (`discover-prd`, `api-contract-extractor`, `gap-analyzer`,
`adr-writer`, `scope-mapper`, `jira-backlog-builder`) are implemented and app-agnostic,
and now carry a `## Governance` preamble binding them to the SAGE Constitution
(`constitution/SAGE-CONSTITUTION.md`). Informed by three QuestIC Discovery runs (QIMS
epic QW-616, Survey Reminders epic QW-746, Activities V2 epic QW-796) — those runs
predate this kit's conventions (no `Subtype`/`Triggered-by`/`Decided-by` on their ADRs,
no Decisions & Assumptions Register) and are **historical examples, not conformant
reference output**. A conformant synthetic reference set is planned but not yet
present — see `adr-writer`'s "Reference ADRs" table for the current (pre-cleanup) state
of that gap. No tag has been cut yet; consuming repos currently pin to a commit SHA.

## Reference implementation

Glue (`apps/quest-ic/glue`) is the reference implementation of the Delivery-side
adapter contract (`docs/specs/testing/architecture.md`) and was the origin of this
kit's conventions. `apps/quest-ic/glue/docs/specs/incentives/` holds one historical
Discovery run (QIMS, pre-dates the current skill conventions).
