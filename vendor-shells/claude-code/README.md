# Claude Code vendor shell for `discovery-lead`

Claude Code discovers skills from `.claude/skills/`, **not** `.agents/skills/` — so the
seven skills `discovery-lead` invokes (everything in the kit except itself) are
invisible to Claude Code's `Skill` tool by default. This shell exists to compensate for
that gap; read the note inside `discovery-lead.md` about how it does so.

## Setup (required — no zero-duplication option exists for Claude Code)

Copy `discovery-lead.md` into either:
- `<repo-root>/.claude/agents/discovery-lead.md` — project-scoped (recommended; check
  it into version control so the whole team gets it), or
- `~/.claude/agents/discovery-lead.md` — personal, available in every project

Restart Claude Code if `.claude/agents/` did not already exist in this repo before this
session started — the directory watcher only covers directories present at startup.

## Optional: make the seven skills natively discoverable

If you'd rather Claude Code discover the kit's skills via its normal `Skill` tool
(instead of `discovery-lead.md`'s Read-tool workaround), symlink them in:

```bash
mkdir -p .claude/skills
ln -s ../../.agents/skills/_shared/skills/discover-prd .claude/skills/discover-prd
ln -s ../../.agents/skills/_shared/skills/api-contract-extractor .claude/skills/api-contract-extractor
ln -s ../../.agents/skills/_shared/skills/gap-analyzer .claude/skills/gap-analyzer
ln -s ../../.agents/skills/_shared/skills/adr-writer .claude/skills/adr-writer
ln -s ../../.agents/skills/_shared/skills/architecture-writer .claude/skills/architecture-writer
ln -s ../../.agents/skills/_shared/skills/scope-mapper .claude/skills/scope-mapper
ln -s ../../.agents/skills/_shared/skills/jira-backlog-builder .claude/skills/jira-backlog-builder
```

Verify each symlink resolves (`ls -la .claude/skills/`) before relying on it — a broken
symlink fails silently from Claude Code's perspective (the skill just doesn't appear).
With this in place, `discovery-lead.md`'s frontmatter may add:

```yaml
skills:
  - discover-prd
  - api-contract-extractor
  - gap-analyzer
  - adr-writer
  - architecture-writer
  - scope-mapper
  - jira-backlog-builder
```

to preload all seven into the agent's context at startup, per Claude Code's `skills`
frontmatter field. This is optional — the shell's Read-tool workaround works without it.

## Keeping in sync

The shell's pointer text never needs edits. If you added the symlinks above, they
follow the submodule automatically — only the submodule ref needs bumping.
