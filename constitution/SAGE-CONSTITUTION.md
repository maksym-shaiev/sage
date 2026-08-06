# SAGE Constitution

Version: 0.1.0 (tag: `v0.1.0`)
Applies to: every repo that mounts SAGE at `.agents/skills/_shared`

This file is the single, versioned source of governance for AI agents working across
Discovery and Delivery in any repo that consumes SAGE. It replaces per-repo copies of
the same rules — a repo's `AGENTS.md` states the rule headlines and points here for the
full text, rather than duplicating it.

---

## Article 0 — Precedence & Amendment

1. This constitution governs. Where a consuming repo's own documentation conflicts with
   an article here, this constitution wins unless the conflict is a properly declared
   **Extension** (Article VI).
2. A consuming repo MAY declare Extensions that add repo-specific verification steps to
   a Universal (Article I) or Discovery (Article II) rule. An extension MUST NOT weaken,
   contradict, remove, or bypass the rule it extends — it may only add conditions or
   required actions.
3. A consuming repo MUST NOT fork or locally edit this file. If a rule does not fit,
   declare an Extension (Article VI) or raise the change here, in `sage`, for all
   consumers.
4. Amendment happens only in this repository (`sage`), by an explicit human-approved
   change, and ships as part of a released tag. Consuming repos adopt a new version of
   this constitution the same way they adopt a new skill version — by bumping their
   pinned submodule reference.

## Article I — Universal Governance

Applies during **both** Discovery and Delivery, in every repo that mounts SAGE. A
consuming repo may layer repo-specific verification machinery on top of these five
rules by declaring an Extension in its own `AGENTS.md` (Article VI) — this article
states only the universal, stack-agnostic form.

### G1 — Specs Before Code, No Exceptions

Before any investigation, research, or code change — even when given a specific file,
class, or method name — read the repo's spec index (`docs/specs/README.md` by SAGE
convention) first and locate the relevant spec(s). A user-provided code snippet is a
starting hint, not a reason to skip spec discovery. This rule has no exceptions.

### G2 — Undocumented Functionality → Propose Spec

On encountering significant undocumented functionality (features, subsystems,
integrations, non-trivial data flows or query strategies), stop and propose creating a
spec before proceeding. Does not apply to utility functions or private helpers.

### G3 — Code/Spec Mismatch → STOP

If the codebase contradicts a spec:

- **Critical mismatch** (behaviour, API contracts, auth flows): hard stop — produce a
  mismatch report and require human resolution before continuing.
- **Minor mismatch** (naming, formatting, detail gaps): flag it, propose a spec update,
  ask permission to continue.

A repo may declare additional Critical-mismatch conditions as an Extension — see
Article VI, and that repo's own `AGENTS.md` for any extension it has declared.

### G4 — User Request Contradicts Spec → Red Alert

If a user asks you to implement something that directly contradicts a documented spec,
stop immediately and report the conflict. Do not proceed until the spec is updated or
the conflict is explicitly acknowledged.

### G5 — Post-Change Spec Obligation

After completing any code change (bug fix, feature, refactor), you MUST:

- Check whether the change affects any existing spec. If yes, propose an update.
- Check whether the change exposes behaviour that is not yet documented (architectural
  patterns, data flows, business rules, non-trivial query strategies). If yes, propose
  creating a new spec.

A task is not complete until this check is done and either a spec update/creation is
proposed or explicitly declined by the human.

A repo may declare additional triggers and verification steps as an Extension — see
Article VI, and that repo's own `AGENTS.md` for any extension it has declared.

## Article II — Discovery Governance

Applies only while producing Discovery artifacts (discovery brief, architecture spec,
ADRs, scope of work) — i.e. before code exists for the initiative in question.

**Severity key:**
- **[HARD STOP]** — the agent must halt on violation and cannot proceed without a human
  resolving it in the same turn.
- **Advisory** — the agent must record the violation (in the artifact's self-check /
  the `discovery-lead` agent's Handoff Criteria) and may proceed; a human reviews it
  before the Discovery Gate (Article IV).

### D1 — No Assumed assertion may be written as fact **[HARD STOP]**

Every non-obvious claim in a Discovery artifact carries a provenance marker (Article
III): Verified or Confirmed. A claim with neither marker is **Assumed** and must not be
written into a spec as if settled. It must instead be recorded as an open entry in the
Decisions & Assumptions Register (Article III) and, where the artifact's coverage rule
requires it, block the item it concerns.

### D2 — Ambiguity → open question, never a plausible default **[HARD STOP]**

Facing a gap, contradiction, or missing information, the agent records an open question.
It does not choose a plausible-sounding default and proceed as though the gap were
closed. "Plausible" is not "Confirmed."

### D3 — Every gap is recorded and closed only with provenance

An open question is closed only by new evidence (→ Verified) or a human decision (→
Confirmed, via an ADR or an inline register entry — Article III). It may not be closed
by the agent's own inference alone, and it may not be silently dropped from the
register.

### D4 — Every decision carries Decided-by, Decided-on, and what it was decided against

Any Confirmed entry in the register, and every ADR, records who decided, when, and which
alternative(s) were rejected and why. "It seemed right" is not a decision record.

### D5 — No invented identifiers

Endpoint paths, field names, class names, database identifiers, and ticket keys must
trace to evidence (live API response, OpenAPI entry, codebase reference, or PRD quote)
or be explicitly marked as proposed/new (`status: new` — Article III). Never presented
as already existing when they are not.

### D6 — Source-of-truth precedence: live API > codebase > PRD

When these disagree: the live API is authoritative for what currently exists; the
codebase is authoritative for behaviour the API cannot express; the PRD is authoritative
only for what is wanted, never for what already exists. A gap between them is a
**finding**, not an error in either source — see `guides/source-of-truth-precedence.md`.

### D7 — The Discovery Gate requires an empty Assumed register **[HARD STOP]**

Discovery may not pass its human-approval gate (Article IV) while the Decisions &
Assumptions Register contains any entry still in the Assumed state. Every entry must be
Verified, Confirmed, or explicitly Open-and-blocking (its dependent scope item marked
Blocked, not silently included in scope).

### D8 — Architecture must satisfy the completeness criterion, or mark items N/A

An architecture spec produced during Discovery must address every item in Article V, or
explicitly state which items are Not Applicable and why. It may not simply omit a
dimension.

## Article III — Provenance Model

Every non-trivial assertion in a Discovery artifact is in exactly one of three states:

| State | Definition | Marker | May be written as settled fact? |
|---|---|---|---|
| **Verified** | Traced to evidence obtained directly — a live API response, an OpenAPI/Swagger entry, a codebase reference (`file:line`), or a verbatim PRD quote | `[Verified: <evidence>]` | Yes |
| **Confirmed** | A human decided it; who and when are recorded | `[Confirmed: <name/role>, <YYYY-MM-DD>]` | Yes, as a decision — link the governing ADR or inline register entry |
| **Assumed** | Neither of the above — the agent's own inference, guess, or "reasonable default" | *(none — must not appear as fact)* | **No** — must be lifted into the Decisions & Assumptions Register as an open entry |

### The Decisions & Assumptions Register

Lives as a section inside `discovery-brief.md` (see `discover-prd` Step 5). One row per
open question or pending decision:

| ID | Statement | State | Evidence / Decided-by (+date) | Blocks | Resolution |
|---|---|---|---|---|---|
| OQ-001 | ... | Assumed → Open | — | ITEM-04 | pending |
| OQ-002 | ... | Confirmed | PO — 2026-08-10 | — | inline (no ADR needed) |
| OQ-003 | ... | Confirmed | — | — | ADR-002 |

An entry graduates out of Assumed by one of two paths:

- **Inline Confirmed** — a small decision, recorded directly in the register with
  decider + date, no ADR required (see `adr-writer`'s "Do not create an ADR for" list
  for what stays inline).
- **ADR** — a significant decision (per `adr-writer`'s trigger list) is written as a
  full ADR; the register row's Resolution links to it.

### Contract-level provenance

Endpoint/contract claims use the same three states via a `status` tag rather than a
register row: `existing` (Verified against a live response) · `extension` (Verified
base path, Confirmed addition) · `new` (Confirmed-as-proposed — it does not exist yet,
so it is anchored by an owning scope item instead of a live response) · `internal`
(this repo only, no upstream dependency exists to verify against). See
`api-contract-extractor` for the full convention.

## Article IV — The Discovery Gate

Discovery produces three artifacts — architecture, ADR(s), scope of work — behind one
gate. The gate passes only when **all** of the following hold:

1. The Decisions & Assumptions Register contains zero entries in the Assumed state (D7).
2. Every register entry is Verified, Confirmed, or Open-and-blocking (its scope item
   marked Blocked).
3. The architecture spec satisfies Article V, or explicitly marks N/A items with a
   reason.
4. Every scope item traces to a covered, deferred, or blocked requirement — none
   silently dropped.
5. A human has explicitly reviewed and approved the artifact set in this form.

**Recording approval:** state it directly in the discovery brief's own Handoff note —
who approved, on what date, and against which artifact version (commit reference if
available). No separate manifest file is required at this time; if one is introduced
later, this article will be updated to point at it rather than duplicate the record.

## Article V — Architecture Completeness Criterion

A Discovery architecture spec must cover — or explicitly mark **Not Applicable** with a
one-line reason — each of:

1. **Component map** — new components, changed components, untouched components,
   explicitly named
2. **Sequence diagram** — one per user-facing or system-facing flow
3. **Data model changes** — new/changed entities, fields, relationships
4. **Integration points** — each with its owning lane/team
5. **Auth model** — who can call what, under what token/session
6. **Failure modes** — what happens when a dependency is unavailable or rejects the
   request
7. **Data ownership** — which system is authoritative for which data
8. **Constraints and non-goals** — explicitly stated, not left implicit

## Article VI — Repo Extension Slots

A consuming repo may extend Article I with repo-specific verification machinery,
declared in its own `AGENTS.md` under a **"SAGE Extensions"** heading. An extension:

- Names the article and rule it extends (e.g. "extends G3")
- States the additional condition and the additional required action
- MUST NOT weaken, contradict, remove, or bypass the rule it extends — it only adds

Extensions do not require a change to this constitution. They are declared and read
entirely within the declaring repo's own `AGENTS.md` — this constitution deliberately
holds **no registry of them**. Registering repo-specific extensions here would load
every consumer's agents with every other consumer's stack-specific tooling, and would
force a constitution version bump (and a re-pin by every consumer) whenever any one
repo changed its own local verification machinery — precisely the cross-repo coupling
SAGE exists to remove.

**Example (illustrative, not a real repo):** a repo whose CI generates typed API
clients from its OpenAPI spec might extend G3 with *"a stale generated client (older
than the last committed OpenAPI change) is an additional Critical mismatch — hard stop
until the client is regenerated."* That text lives only in that repo's `AGENTS.md`.

## See Also

- `skills/discover-prd/SKILL.md` — Step 5 assembles the Decisions & Assumptions Register
- `skills/adr-writer/SKILL.md` — ADR structure implementing D4 (Decided-by / Decided-on)
- `skills/api-contract-extractor/SKILL.md` — `status` tag implementing Article III's
  contract-level provenance
- `guides/source-of-truth-precedence.md` — full precedence rule referenced by D6
- `README.md` — how a repo mounts SAGE and inherits this constitution
