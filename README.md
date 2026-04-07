# Hermes Agent Skills

Shareable skills for [Hermes](https://hermes-agent.nousresearch.com/) — published by [keepnotes.ai](https://keepnotes.ai).

## Skills

| Skill | Category | Description |
|-------|----------|-------------|
| [github-vault-search](skills/github/github-vault-search/SKILL.md) | github | Search GitHub for Obsidian vaults by topic. Multi-strategy search, vault verification, quality sampling, nightly pull via cron. Includes `obsidian-search` script. |
| [repo-intake-check](skills/github/repo-intake-check/SKILL.md) | github | Safety scanner for untrusted cloned repos. Checks hooks, symlinks, Obsidian plugins, autorun configs, agent prompt injection. Outputs CLEAN / REVIEW / CAUTION. Includes `repo-intake-check` script. |
| [keep-research-workflow](skills/note-taking/keep-research-workflow/SKILL.md) | note-taking | Use Keep as a research OS: collect URLs and PDFs, promote to long-form notes, cross-link findings, and surface knowledge via semantic search. |

## Using this repo with Hermes

Add as a skill source:

```bash
hermes skills tap add keepnotes-ai/hermes-skills
```

Then browse, search, or install skills with the Hermes skills hub.

## Keep Memory Plugin

Keep is a memory plugin for Hermes Agent. Not yet in core, but you can check out and run Hermes from [this PR branch](https://github.com/NousResearch/hermes-agent/pull/5172), then run `hermes memory setup` and configure Keep with local or API-based models.

## Layout

```
skills/<category>/<skill-name>/
  SKILL.md          # skill definition (frontmatter + instructions)
  scripts/          # bundled scripts referenced by the skill
  references/       # reference docs loaded on demand
  templates/        # reusable templates
```
