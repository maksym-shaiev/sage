# ai-sdd-kit — Install & Handoff

This kit was extracted locally at `maru/libs/ai-sdd-kit`. The steps below require Bitbucket
org access and are intended to be run by a human. Until they are done, apps can consume the
kit via a local symlink (dev-only).

## 1. Create the remote repo (human, Bitbucket access required)

```bash
# Create an empty repo in the stagwellmc org, e.g. "ai-sdd-kit", then:
cd /home/max/maru/libs/ai-sdd-kit
git init
git add .
git commit -m "Initial ai-sdd-kit: SDD workflow core (guides, templates, core skills)"
git branch -M main
git remote add origin git@bitbucket.org:stagwellmc/ai-sdd-kit.git
git push -u origin main
```

## 2. Wire into an app as a submodule (preferred)

```bash
cd <app-repo-root>          # e.g. apps/quest-ic/glue
git submodule add git@bitbucket.org:stagwellmc/ai-sdd-kit.git .agents/skills/_shared
git commit -m "Add ai-sdd-kit shared SDD workflow as submodule"
```

App-local skills then reference `.agents/skills/_shared/guides/*` and
`.agents/skills/_shared/templates/*`.

## 3. Local symlink alternative (dev-only, before the remote exists)

```bash
cd <app-repo-root>
ln -s ../../../../libs/ai-sdd-kit .agents/skills/_shared   # adjust depth per app location
# do NOT commit the symlink; replace with the submodule once the remote exists
```

## 4. De-duplicate the Glue copies (after submodule is wired)

The core artifacts currently exist both in Glue (`docs/specs/process/` +
`.agents/skills/api-contract-extractor`, `gap-analyzer`) and in this kit. Once the submodule
is mounted, decide per artifact:

- **Guides + templates**: keep the canonical copy in the kit; in Glue, replace the
  `docs/specs/process/` guide/template files with short pointers to
  `.agents/skills/_shared/guides|templates/...`, OR leave Glue's copies as the reference
  implementation and treat the kit as the distributable. Recommended: pointer files in Glue
  to avoid drift.
- **Core skills** (`api-contract-extractor`, `gap-analyzer`): remove the Glue-local copies
  once `_shared` is mounted; opencode discovers skills under `_shared`.
- **App skills** (`discover-prd`, `scope-mapper`, `jira-backlog-builder`): stay Glue-local;
  no change.

> Do this de-duplication as a deliberate follow-up commit, not automatically — verify skill
> discovery works from `_shared` first.

## 5. Verify

- `make` / opencode picks up skills from `_shared`.
- Run the equivalence check in `vendor-tooling-guide.md` on the QIMS reference PRD.

## Current state (as extracted)

- Kit content is complete and self-contained at `maru/libs/ai-sdd-kit`.
- No remote repo yet; no submodule wired.
- Glue retains its own authoritative copies (nothing in Glue was removed).
- This is fully reversible: deleting `maru/libs/ai-sdd-kit` leaves Glue untouched.
