# OpenCode vendor shells for `discovery-lead`

Both shells register the same agent; pick one. Neither contains any logic — they are
thin pointers over `skills/discovery-lead/SKILL.md`, per `../../vendor-tooling-guide.md`'s
"no logic in shells" rule.

## Option A — `opencode.json` (recommended: zero duplication)

Uses opencode's `{file:...}` prompt reference, so the role definition is loaded
directly from the submodule and never copied into the consuming repo at all.

- If the repo has **no** `opencode.json` yet: copy `opencode.json.example` to
  `<repo-root>/opencode.json`.
- If the repo **already has** an `opencode.json`: merge only the `agent.discovery-lead`
  block from `opencode.json.example` into the existing file's `agent` object — do not
  overwrite the rest of the file.

The `{file:...}` path (`./.agents/skills/_shared/skills/discovery-lead/SKILL.md`) is
relative to the config file's location, so it only resolves correctly when
`opencode.json` sits at the repo root and the SAGE submodule is mounted at
`.agents/skills/_shared` (the standard mount point — see `../../INSTALL.md`).

Verify after copying: `opencode` should list `discovery-lead` as a primary agent
(cycle with **Tab**, or `@discovery-lead` to mention it directly).

## Option B — markdown agent file

Copy `discovery-lead.md` into either:
- `<repo-root>/.opencode/agents/discovery-lead.md` — project-scoped, or
- `~/.config/opencode/agents/discovery-lead.md` — available in every project on this
  machine

Unlike Option A, a markdown agent's body **is** its system prompt — there is no
`{file:...}` equivalent inside markdown frontmatter. So this shell's body is a short
pointer instruction ("read and follow `.agents/skills/_shared/skills/discovery-lead/
SKILL.md`") rather than a true zero-duplication reference. Prefer Option A when
possible; use Option B only if the repo's conventions favor a `.opencode/agents/` file
over `opencode.json` edits.

## Keeping in sync

Neither shell needs edits when the kit updates — Option A always reads the latest
`SKILL.md` from wherever the submodule is pinned; Option B's pointer text never changes
either. Bumping the submodule ref is the only sync step, for both options.
