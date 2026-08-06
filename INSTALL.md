# SAGE — Install & Handoff

This kit is published at `git@github.com:maksym-shaiev/sage.git`. It ships the
**Discovery phase only** — guides, templates, and all six Discovery-pipeline skills.
Delivery-phase enforcement (spec schema, CI audit, test generation) is per-app and not
part of this kit; see `README.md`.

## 1. Wire into an app as a submodule (the standard way — required for committed work)

```bash
cd <app-repo-root>          # e.g. apps/quest-ic/glue or apps/kato
git submodule add git@github.com:maksym-shaiev/sage.git .agents/skills/_shared

# pin to a released tag once one exists (see README "Status" for the current tag)
cd .agents/skills/_shared && git checkout <tag> && cd -

git add .gitmodules .agents/skills/_shared
git commit -m "Add SAGE (Discovery phase) as a pinned submodule"
```

The agent discovers all skills under `.agents/skills/_shared/skills/*/SKILL.md`
automatically. App-local skills may reference `.agents/skills/_shared/guides/*` and
`.agents/skills/_shared/templates/*`.

Updating the pin later:

```bash
cd .agents/skills/_shared && git fetch --tags && git checkout <new-tag> && cd -
git add .agents/skills/_shared
git commit -m "Bump SAGE to <new-tag>"
```

## 2. Local symlink (dev-only — solo iteration, never committed)

```bash
cd <app-repo-root>
ln -s <absolute-or-relative-path-to>/libs/sage .agents/skills/_shared
# do NOT commit this symlink; it cannot be shared with another developer.
# Replace with the submodule (step 1) for any work meant to be shared or committed.
```

## 3. Verify

- The agent (or `ls .agents/skills/_shared/skills/`) lists all six skills:
  `discover-prd`, `api-contract-extractor`, `gap-analyzer`, `adr-writer`, `scope-mapper`,
  `jira-backlog-builder`.
- No app should keep a local copy of a core skill — core skills are consumed only
  through `_shared`. If an app-local skill directory shares a name with a core skill,
  remove the local copy; the shared one supersedes it.

## Current state

- Kit content is versioned at `git@github.com:maksym-shaiev/sage.git`, branch `main`.
- No released tag yet — pin instructions above will name one once cut.
- Consuming apps and their current mount mechanism:
  - **Glue** (`apps/quest-ic/glue`): `.agents/skills/_shared` is currently a local,
    untracked symlink to `libs/sage` — pending migration to the pinned submodule form
    (step 1) once a tag is cut.
  - **Kato** (`apps/kato`): not yet mounted.
- This is fully reversible: removing the submodule/symlink and `libs/sage` leaves each
  app's own `docs/specs/**` untouched — this kit only ever supplies skills, guides, and
  templates, never app content.
