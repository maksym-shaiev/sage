# SAGE — Install & Handoff

This kit is published at `git@github.com:maksym-shaiev/sage.git`. It ships the
**Discovery phase only** — guides, templates, and all eight Discovery-pipeline skills
(the six original skills plus `architecture-writer` and the `discovery-lead`
orchestrator). Delivery-phase enforcement (spec schema, CI audit, test generation) is
per-app and not part of this kit; see `README.md`.

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

- The agent (or `ls .agents/skills/_shared/skills/`) lists all eight skills:
  `discover-prd`, `api-contract-extractor`, `gap-analyzer`, `adr-writer`,
  `architecture-writer`, `scope-mapper`, `jira-backlog-builder`, `discovery-lead`.
- No app should keep a local copy of a core skill — core skills are consumed only
  through `_shared`. If an app-local skill directory shares a name with a core skill,
  remove the local copy; the shared one supersedes it.

## 4. Configuring the discovery-lead agent (optional)

`discovery-lead` works as a plain skill with no agent configuration at all — invoke it
by name on any vendor that discovers skills from `.agents/skills/_shared/skills/`, and
it sequences `discover-prd` → `adr-writer` → `architecture-writer` → `scope-mapper` per
its own `SKILL.md`. An agent shell on top of it adds an `@`-mention entry point and an
isolated context window; it adds no capability the skill doesn't already have. Canonical
shell text ships in `vendor-shells/` so it travels with the submodule pin — copy it in
rather than hand-writing your own.

### OpenCode CLI

Skills are auto-discovered from `.agents/skills/**` — nothing extra is needed for the
skill itself; verify with `ls .agents/skills/_shared/skills/`.

For the optional agent, see `vendor-shells/opencode/README.md` for both setup options
(a `{file:...}`-referencing `opencode.json` block with zero duplication, or a
`.opencode/agents/discovery-lead.md` pointer file). Once configured:

```bash
opencode          # then Tab to cycle to discovery-lead, or @discovery-lead to mention it
```

### Claude Code CLI

Claude Code discovers skills from `.claude/skills/`, **not** `.agents/skills/` — the
eight SAGE skills are not auto-discovered. See `vendor-shells/claude-code/README.md`:

1. Copy `vendor-shells/claude-code/discovery-lead.md` into `.claude/agents/` (project)
   or `~/.claude/agents/` (personal). Restart Claude Code if `.claude/agents/` did not
   already exist in this session.
2. Optionally symlink the eight kit skills into `.claude/skills/` (instructions in that
   README) so Claude's native `Skill` tool can find them too — without this, the shell's
   pointer body instructs Claude to `Read` each `SKILL.md` by path instead, which works
   but doesn't benefit from Claude's skill-preloading.

```text
Use the discovery-lead agent to start Discovery for this initiative: <PRD link>
```

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
