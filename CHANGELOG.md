# Changelog

Adoption history and provenance for the kit and constitution. The constitution itself
(`constitution/SAGE-CONSTITUTION.md`) stays normative and app-agnostic — it does not
carry this history; this file does.

## Unreleased

### Constitution introduced (v0.1.0, pre-tag)

- **Origin of Article I (Universal Governance, G1–G5):** extracted from the
  "Documentation Governance" sections independently written in Glue's
  (`apps/quest-ic/glue/AGENTS.md`) and Kato's (`apps/kato/AGENTS.md`) `AGENTS.md`
  files. G1, G2, and G4 were identical between the two repos. **G3 and G5 had already
  drifted**: Glue's G3 carried an additional `.hurl`-coverage Critical-mismatch
  condition Kato's did not have, and Glue's G5 carried ~40 lines of two-trigger
  verification machinery (API Contract Trigger, Behavior Trigger — regenerating
  `public/openapi.yaml`, invoking `contract-drift`/`test-author`, running
  `make e2e-audit`) that Kato's G5 did not have. This drift, discovered before any
  constitution existed, is the concrete evidence that motivated centralising the
  universal rules in one versioned place rather than letting each repo maintain its
  own copy.
- Glue's drifted G3/G5 additions became the model for **Article VI (Repo Extension
  Slots)** — the mechanism by which a repo may add stack-specific verification without
  forking the universal rule. Glue's own extension text was rewritten to live entirely
  in Glue's `AGENTS.md`; the constitution names the mechanism only, with a generic
  illustrative example — it does not register any specific repo's extension. (An
  earlier draft of Article VI briefly registered Glue's extension text in full; this
  was corrected before the first tag because it re-introduced the coupling — one
  repo's tooling loaded into every consumer's context, and a constitution version bump
  forced on every consumer whenever Glue's local machinery changed.)
- **Origin of Article II (Discovery Governance, D1–D8):** new rules, not extracted from
  either repo. Motivated by auditing three real Discovery runs (QIMS epic QW-616,
  Survey Reminders epic QW-746, Activities V2 epic QW-796) and finding zero of the
  three artifact sets carried any record of *why* a decision was made, *who* made it,
  or *which alternative was rejected* — decisions and unresolved gaps were
  indistinguishable from the agent's own inference. D1/D2/D7 are hard stops for this
  reason; D3–D6/D8 are advisory pending a deterministic audit (not yet built).
- **`AGENTS.md` rewrites:** both Glue's and Kato's governance sections were rewritten
  from full inline rule text to rule headlines + a pointer to this constitution. Fixed
  a pre-existing bug found in Glue's file in the same pass: a stray, truncated
  duplicate of rule 5 (Post-Change Spec Obligation) sitting between the Local AI
  skills table and the Quick Reference section.

### `architecture-writer` and `discovery-lead` added; Article V reconciled against real prior art

- **Kato's `sage-delivery` skill (Step 1) was the closest existing thing to an
  architecture-writing skill** — but audited against Article V's 8-point completeness
  criterion, it covered only 5 of 8: no `## Integration Points` (with owning lane), no
  `## Auth Model`, and no `## Failure Modes` sections existed anywhere in its required
  section list. Its Mermaid template also hard-coded `Kato REST API`/`PostgreSQL` as
  participant names — the same category of app-specific leakage fixed elsewhere in this
  kit (see the constitution's Article VI cleanup and `be-task.md`'s genericisation).
  `architecture-writer` reconciles rather than copies: keeps what Step 1 got right
  (reading the topology doc before drawing, the metadata header shape, requiring a
  sequence diagram per flow) and adds the 3 missing sections plus a genericised
  template.
- **`discovery-lead` orchestrates `discover-prd` → `adr-writer` →
  `architecture-writer` → `scope-mapper`** as a single entry point, with an explicit
  ownership boundary between itself (ADRs for brief-flagged decisions) and
  `scope-mapper`'s own Step 3b (ADRs surfacing only during scoping) — the two do not
  compete for the same decisions. Its Handoff Criteria self-check covers the *complete*
  four-artifact set (cross-artifact consistency: ADR↔scope-item back-links, lane
  agreement between architecture and scope, no orphaned proposed endpoints), which no
  single skill could check in isolation.
- **`reference/example-initiative/` gained `architecture.md`**, closing a gap the set's
  own README had explicitly flagged as deliberately omitted one phase earlier. It
  demonstrates all 8 Article V dimensions, including one explicit `N/A` (no cross-repo
  integration applies to this single-service toy initiative) — the point being that D8
  requires *stating* non-applicability, not that every dimension must be substantial.
- **Vendor shells for `discovery-lead` ship in `vendor-shells/`, not as prose recipes.**
  OpenCode's `{file:...}` prompt reference achieves genuine zero-duplication; Claude
  Code has no equivalent (a markdown agent's body *is* its prompt) and additionally does
  not discover skills from `.agents/skills/` at all, so its shell also carries an
  explicit workaround note and an optional symlink recipe. This is the one skill in the
  kit with a shipped shell — the other seven are invoked by name, per
  `vendor-tooling-guide.md`'s "generate on demand" principle.
