---
name: github-vault-search
description: Search GitHub for Obsidian vaults (or any topic-tagged repos) by subject keyword. Covers multi-strategy API search, vault confirmation via directory check, and quality sampling via clone+analyze.
triggers:
  - "find obsidian vault"
  - "search github for vaults"
  - "obsidian vault about X"
  - "find repos about topic on github"
---

# GitHub Vault / Topic Repo Search

Use when the user wants to find Obsidian vaults (or any repos) on GitHub by subject — e.g. "is there a gardening vault on GitHub?"

Obsidian can open and edit vaults, and Keep can index them.

## Key Lessons Learned

- `gh search repos --json` does NOT support a `topics` field — omit it or use `--topic` flag
- `gh search repos "keyword"` free-text returns empty results for niche combos — use `gh api` directly
- `.obsidian` verification via `gh api repos/{owner}/{repo}/contents` only checks repo root — nested vaults are missed, but root is the common case
- Background `&` clones in a single terminal command don't persist between terminal() calls — clone sequentially or use `wait` within the same shell invocation
- Verify top 10 candidates, not just 5 — good vaults often rank 6-10 due to low star counts

## Search Strategies (run all three, deduplicate by full_name)

### 1. GitHub topic tags (most precise)
```bash
gh api "search/repositories?q=topic:obsidian-vault+topic:{slug}&sort=stars&per_page=10" \
  --jq '.items[] | {name: .full_name, stars: .stargazers_count, desc: .description, url: .html_url}'
```

### 2. obsidian + keyword in name/description
```bash
gh api "search/repositories?q=obsidian+{keyword}+in:name,description&sort=stars&per_page=10" \
  --jq '.items[] | ...'
```

### 3. keyword + vault/notes in name/description (catches untagged vaults and personal learning repos)
```bash
# "vault" variant
gh api "search/repositories?q={keyword}+vault+in:name,description&sort=stars&per_page=10" \
  --jq '.items[] | {name: .full_name, stars: .stargazers_count, desc: .description, url: .html_url}'

# "notes" variant — catches personal learning journals and study note repos
gh api "search/repositories?q={keyword}+notes+in:name,description&sort=stars&per_page=10" \
  --jq '.items[] | {name: .full_name, stars: .stargazers_count, desc: .description, url: .html_url}'
```

### 4. topic hyphenated in name/topics (surfaces code ecosystems + any tagged notes)
```bash
gh api "search/repositories?q={hyphenated-keyword}+in:name,topics&sort=stars&per_page=15" \
  --jq '.items[] | {name: .full_name, stars: .stargazers_count, desc: .description, url: .html_url}'
```
This also tells you whether the topic has a rich *code* presence with zero vault presence — a "gap signal" worth reporting to the user.

## Confirm It's a Real Vault

Check for `.obsidian` directory at repo root:
```bash
gh api "repos/{owner}/{repo}/contents" --jq '.[].name' | grep -q '.obsidian' && echo "confirmed"
```

Do this for top 10 candidates. Repos without `.obsidian` at root may still be vaults (published with Quartz/mkdocs) — list them as "unconfirmed candidates" not as failures.

If `.obsidian` is absent, do a second check — inspect the subfolder structure and count `.md` files:
```bash
gh api "repos/{owner}/{repo}/git/trees/HEAD?recursive=1" \
  --jq '.tree[] | select(.path | endswith(".md")) | .path' | head -20
```
A repo with organized folders of `.md` files (especially with Summary, Q&A, or ROADMAP files) is a real notes collection even without Obsidian markers. These are often personal study journals or curriculum notes — still worth cloning and pointing Keep at the content subfolder.

## Reusable Script

Bundled at `scripts/obsidian-search` in this skill. Install once:
```bash
cp <skill-dir>/scripts/obsidian-search ~/bin/obsidian-search
chmod +x ~/bin/obsidian-search
```

Usage:
```bash
python3 ~/bin/obsidian-search gardening
python3 ~/bin/obsidian-search philosophy
python3 ~/bin/obsidian-search "machine learning"
```

Runs all 4 strategies, deduplicates, verifies top 10, prints confirmed vaults first.

## Full Workflow: Search → Sample → Secure → Store → Index

### 1. Confirm destination with user before cloning
Suggest: `~/Documents/Vaults/community/<topic>/`
Ask explicitly — do not assume or default silently.

### 2. Clone with depth=1
```bash
git clone --depth=1 https://github.com/{owner}/{repo}.git \
  ~/Documents/Vaults/community/<name>
```

### 3. Run security check
```bash
python3 ~/bin/repo-intake-check ~/Documents/Vaults/community/<name>
```
See the `repo-intake-check` skill for threat model. Key vault-specific risk: bundled Obsidian plugins (`.obsidian/plugins/*.js`) run as JS inside Obsidian — check plugin names against the official list.

### 4. Index into keep (per-vault, with tags)
```bash
keep put ~/Documents/Vaults/community/<name> -r --watch \
  -t topic=<topic> -t source=community
```

The `--watch` flag is intentional: keep monitors the vault directory for changes. Combined with a nightly `git pull` (see step 6), new notes surface in keep automatically without re-indexing.

**Do NOT index the parent `~/Documents/Vaults/` directory** — keep has a default 1000-file limit and will refuse. Index each vault subdirectory separately. This is better for tagging anyway.

### 5. (optional) Quality sampling — before step 2 if uncertain
```bash
cd ~/tmp && git clone --depth=1 https://github.com/{owner}/{repo}.git vault-sample
```

Analyze quality (see Quality Sampling section below), then clone permanently only if it passes.

### 6. Schedule nightly git pull via cron

Once a vault is cloned and indexed with `--watch`, set up a cron job to pull updates nightly:

```bash
# Pull all community vaults every night at 2am
# keep's --watch watcher picks up any changed files automatically
0 2 * * * cd ~/Documents/Vaults/community && for d in */; do git -C "$d" pull --ff-only --quiet 2>/dev/null; done
```

Add with `crontab -e`. The `--ff-only` flag skips vaults with local modifications (safe). Failed pulls are silently skipped — check manually if a vault stops updating.

### Quality Sampling (detail for step 5 above)
```bash
cd ~/tmp && git clone --depth=1 https://github.com/{owner}/{repo}.git

Analyze with Python:
```python
from pathlib import Path
import re

root = Path("~/tmp/vault-samples/VaultName").expanduser()
md_files = list(root.rglob("*.md"))
total_chars = sum(len(f.read_text(errors="ignore")) for f in md_files)
wikilinks = sum(len(re.findall(r'\[\[.+?\]\]', f.read_text(errors="ignore"))) for f in md_files)
tags = set()
for f in md_files:
    tags.update(re.findall(r'(?<!\w)#[\w/-]+', f.read_text(errors="ignore")))

print(f"{len(md_files)} files, {total_chars:,} chars, {wikilinks} wikilinks, {len(tags)} unique tags")
```

### Quality signals
- GOOD: >100 .md files, avg >1000 chars/file, many wikilinks (graph structure), specific tags
- OK: 50-100 files, some wikilinks, real topic tags
- WEAK: <30 files, no wikilinks, stubs, or just templates

## Canonical `.ignore` additions for vault indexing

These should be in `keep get .ignore` (update via `cat file | keep put - --id .ignore -f`):

```
# Archives (not indexable)
*.zip *.tar.gz *.tar.bz2 *.gz *.rar *.7z

# Node dependencies (nested installs too)
node_modules/*
*/node_modules/*

# Vault generation scripts (not prose notes)
*.ps1 *.bat

# Obsidian canvas files (JSON diagram format, not prose)
*.canvas
```

Do NOT add `LICENSE`, `LICENSE.md`, `CHANGELOG.md` — meaningful in code repos.

## Pitfalls

- "digital garden" repos (Quartz/MkDocs published) may not have `.obsidian` at root but still contain real vault notes
- "rich code, zero vault" is itself a meaningful result — report it. Topics like compressive sensing or audio DSP have hundreds of implementation repos but no published knowledge bases. That gap is worth surfacing to the user (gap = opportunity).
- Some repos are vault *templates* not actual knowledge (check for placeholder content)
- Stars correlate poorly with content quality for personal vaults — a 0-star vault can be excellent
- `gh api` paginates at 10/30/100 per_page; add `&page=2` if results seem thin
- `keep put <parent-dir> -r` will fail if the tree exceeds 1000 files — index each vault subdirectory separately
- `keep put <file> --id .ignore -f` fails with "internal server error" for system docs when using file mode — use stdin: `cat file | keep put - --id .ignore -f`
- keep `.ignore` uses Python fnmatch: `*` matches `/` so `*.pyc` catches nested paths, but `__pycache__/*` only matches top-level `__pycache__` dirs — use `*.pyc` not `__pycache__/*` for reliable coverage
- Vault-specific `.ignore` additions worth adding: `*.zip`, `*.canvas`, `*.ps1`, `*/node_modules/*`
- Do NOT add `LICENSE`, `LICENSE.md`, or `CHANGELOG.md` to `.ignore` — they carry real meaning in code projects. Only exclude them if you're in a pure notes-only context and have confirmed they're always boilerplate.
