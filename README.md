# Hermes Agent Skills

This repository contains shareable skills for [Hermes](https://hermes-agent.nousresearch.com/).

Current skills:
- `note-taking/keep-research-workflow`

## Using this repo with Hermes

Add the repo as a skill source:

```bash
hermes skills tap add keepnotes-ai/hermes-skills
```

Then browse, search, or install skills from it with the Hermes skills hub.

## Memory Plugin

Keep is a memory plugin for Hermes Agent.  Not yet in core, but you can check out and run Hermes from [this PR branch](https://github.com/NousResearch/hermes-agent/pull/5172), then run `hermes memory setup` and configure Keep with local or API-based models.

## Layout

Hermes skills are stored under `skills/<category>/<skill-name>/SKILL.md`.
