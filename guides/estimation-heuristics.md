# Estimation Heuristics
Type: Guide
Status: Draft
Updated: 2026-06-12
Description: Heuristics for estimating AI-assisted BE/FE delivery from a scope of work,
  including AI-assist factors, complexity signals, buffer, and parallel-timeline method.
---

## Overview

This guide makes scope-of-work estimates consistent and defensible. Estimates are
**heuristics**, not commitments — high-uncertainty items get a spike, not a guess.

## AI-assist factors

Adjust raw manual estimates by the expected AI efficiency gain, which differs by discipline:

| Discipline | AI gain vs unassisted | Rationale |
|---|---|---|
| Backend | ~50–60% faster | Specs + contracts fully defined; established client/State patterns to mirror |
| Frontend | ~25–30% faster | UI/UX needs hand-crafting, design fidelity, component work |

Stronger AI gain requires: a defined contract, an existing pattern to follow, and clear
acceptance criteria. Absent those, reduce the assumed gain.

## Complexity signals (per item)

Use the State-implementation approach as the primary backend complexity signal:

| Signal | Complexity | Typical BE size |
|---|---|---|
| Generic provider/processor chain (no custom State class) | Low | 0.5–1.5 days |
| Single custom State class, one upstream call | Medium | 1–2 days |
| Composite custom State (multiple upstream calls merged) | Medium-High | 2–3 days |
| Multi-step flow (e.g. provisioning) or legacy-codebase change | High | 3–5 days + spike |

Frontend complexity signal: number of screens/steps and statefulness (a 3-step wizard >
a single table > a read-only card).

## Buffer and spikes

- Add a **~15% buffer** for integration, review, and QA across the whole estimate.
- For any **High** item touching an unfamiliar/legacy codebase, add a **1-day spike**
  before committing the number (e.g. the Kato sync adapter in the QIMS run).
- Items **blocked** by an open question are **not estimated** until the question resolves.

## Timeline method (parallel BE + FE)

- Estimate BE-days and FE-days per phase separately.
- Phase duration = max(BE-days, FE-days) for that phase (they run in parallel), not the sum.
- The longer discipline is the phase bottleneck; the other can start the next phase's items
  early if dependencies allow.
- Total calendar time = sum of phase durations + buffer.

## Worked example (QIMS reference run)

- 1 BE + 1 FE, AI-assisted, full-time.
- Phase 2 (Admin Core): BE ~13.5 days, FE ~18.5 days → ~4 weeks (FE bottleneck).
- Whole programme: BE ~46 days, FE ~51 days incl. buffer → **~10 weeks** calendar.
- ITEM-00 (Kato sync, High/legacy) flagged for a 1-day spike before its 5-day estimate.
- ITEM-22 (blocked by OQ-2) left unestimated.

## See Also

- `docs/specs/process/ai-sdd-workflow-plan.md` — overall workflow
- `docs/specs/process/_templates/vp-email.md` — estimate communication format
- `docs/specs/incentives/scope-of-work.md` — reference scope that was estimated
