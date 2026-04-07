# Hermes Agent Skills

Shareable skills for [Hermes](https://hermes-agent.nousresearch.com/) — published by [keepnotes.ai](https://keepnotes.ai).

## Skills

| Skill | Category | Description |
|-------|----------|-------------|
| [github-vault-search](skills/github/github-vault-search/SKILL.md) | github | Search GitHub for Obsidian vaults by topic. Multi-strategy search, vault verification, quality sampling, nightly pull via cron. Includes `obsidian-search` script. |
| [repo-intake-check](skills/github/repo-intake-check/SKILL.md) | github | Safety scanner for untrusted cloned repos. Checks hooks, symlinks, Obsidian plugins, autorun configs, agent prompt injection. Outputs CLEAN / REVIEW / CAUTION. Includes `repo-intake-check` script. |
| [keep-research-workflow](skills/note-taking/keep-research-workflow/SKILL.md) | note-taking | Use Keep as a research OS: collect URLs and PDFs, promote to long-form notes, cross-link findings, and surface knowledge via semantic search. |

## Why index Obsidian vaults into Keep?

Once a vault is indexed with `keep put <vault> -r --watch`, its notes become part of
your ambient knowledge context — not just explicitly searchable, but automatically
surfacing alongside your own notes and conversations when semantically relevant.

A community philosophy vault's note on note-taking might appear next to your own
recent reflections on the same topic. A machine learning vault's explanation of a
concept surfaces when you're discussing that concept in conversation. The vault's
knowledge is woven into your assistant's context without you having to ask for it.

Combined with a nightly `git pull` and keep's `--watch` watcher, community vaults
become living feeds — new notes added upstream appear in your context automatically.

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
