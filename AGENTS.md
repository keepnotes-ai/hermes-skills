# Agent Instructions — hermes-skills repo

This file is read by AI agents (Claude Code, Codex, Cursor, etc.) working on this repo.
Follow these conventions exactly.

## What this repo is

A curated set of **published, public** Hermes skills maintained by keepnotes.ai.
Not every skill in `~/.hermes/skills/` belongs here — only skills that are
polished enough to share and useful outside a single setup.

## Adding a skill

1. Create `skills/<category>/<skill-name>/SKILL.md` with valid YAML frontmatter
   (name, description, triggers[]).
2. Add any bundled scripts to `skills/<category>/<skill-name>/scripts/`.
3. Update the README.md skills table (see below).
4. Commit with message: `Add <skill-name> skill`

## Removing or renaming a skill

1. Remove or move the skill directory.
2. Update the README.md skills table.
3. Commit with message: `Remove <skill-name> skill` or `Rename <old> -> <new>`

## README skills table — keep this in sync

The table in README.md must reflect the current contents of `skills/` exactly.
Run this to audit it any time you're unsure:

```bash
find skills -name "SKILL.md" | sort
```

Table format:
```
| [<skill-name>](skills/<category>/<skill-name>/SKILL.md) | <category> | <one-line description> |
```

The description should be taken from (or consistent with) the `description:` field
in the skill's YAML frontmatter. Keep it to one sentence, under ~120 chars.

## Commit conventions

- `Add <skill-name> skill` — new skill
- `Update <skill-name>: <what changed>` — skill content change
- `Fix <skill-name>: <what>` — bug fix in skill or script
- `Remove <skill-name> skill` — skill removed
- `docs: <what>` — README or documentation only
- No merge commits — rebase before push

## What NOT to do

- Do not add skills that contain personal config, API keys, or machine-specific paths.
- Do not add skills that are unfinished drafts (no triggers, placeholder content).
- Do not commit `.hub/`, `.bundled_manifest`, or any Hermes runtime state — these
  are in .gitignore for a reason.
- Do not bulk-import all skills from a local ~/.hermes/skills/ directory.
  Each skill added here is a deliberate publishing decision.
