---
description: Recap of the last 7 days of changes — a detailed technical section plus a non-technical summary. Flags — --prs (include GitHub PRs via gh), --save (markdown export), --days N, fr|en.
argument-hint: [--prs] [--save] [--days N] [fr|en]
allowed-tools: Bash(git rev-parse:*), Bash(git log:*), Bash(git shortlog:*), Bash(git diff:*), Bash(git branch:*), Bash(git remote:*), Bash(command -v gh:*), Bash(gh auth status:*), Bash(gh pr list:*), Bash(date:*), Write
---

Generate a recap of the repository's changes over a recent window (7 days by default), in **two parts**: a detailed technical recap, then a non-technical summary. The goal is for a developer to get the useful detail, and for a non-technical reader (manager, client) to understand it without jargon.

## Arguments

Raw arguments: `$ARGUMENTS`

Parse the following flags (order doesn't matter):
- `--prs` or `--pr` → include GitHub Pull Requests. **Requires `gh` installed and authenticated.**
- `--save` or `--file` → also write the recap to a markdown file.
- `--days N` → replace the 7-day window with N days.
- `fr` / `en` → output language. Default: **the user's language in this conversation**, otherwise **French**.

## 1. Collect the data (git)

First, verify you're inside a git repo: `git rev-parse --is-inside-work-tree`. If not, stop and say so clearly.

Define the window `SINCE`: `7 days ago` by default (or `N days ago` if `--days N`).

Run these commands (adapt as needed):
- `git log --since="$SINCE" --date=short --pretty=format:'%h%x09%ad%x09%an%x09%s'` — commit list (hash, date, author, subject).
- `git log --since="$SINCE" --shortstat --pretty=format:'%h %s'` — change volume per commit (files, +/- lines) to aggregate.
- `git log --since="$SINCE" --name-only --pretty=format:''` — touched files, to group by module/folder.
- `git shortlog -sn --since="$SINCE"` — contributors and commit counts.
- `git branch --show-current` and `git remote get-url origin` — context (branch, repo).

**If there are no commits in the window**: say so clearly and don't fabricate anything. Suggest `--days N` to widen the window.

## 2. GitHub Pull Requests (only if `--prs`)

First check availability: `command -v gh` then `gh auth status`.
- If `gh` is missing or not authenticated: **flag it** ("install and log in to `gh` — `gh auth login` — to include PRs") and **continue without the PRs**, without blocking the rest.

Otherwise, compute the start date (macOS: `date -v-7d +%F`; Linux: `date -d '7 days ago' +%F`; adapt for `--days N`), then:
- `gh pr list --state merged --search "merged:>=<DATE>" --json number,title,author,mergedAt,labels`
- `gh pr list --state open --json number,title,author,createdAt`

## 3. Output — two parts

Write in the chosen language. **Don't just copy the commit list: group and interpret** the meaning of the changes from commit subjects and touched files.

### Part A — Detailed technical recap (for a developer)
- **Header**: repo, branch, actual period covered (dates), number of commits, number of touched files, added/removed lines, contributors.
- **Grouped changes** by theme / module / feature. For each group: what changed and, where deducible, why.
- **Points of attention**: new modules or dependencies, structural refactors, breaking changes, schema/DB migrations, important fixes, technical debt introduced.
- If `--prs`: **merged PRs** (`#num — title — author`) and **still-open PRs** (work in progress).

### Part B — Non-technical summary (for a manager / client)
- 3 to 6 bullets in **plain language**, framed as **value and impact**: what was **added**, **improved**, **fixed**.
- **Zero jargon**: no file names, function names, or technical terms. A non-technical person must understand every bullet.

## 4. Export (only if `--save`)

Write both parts to `weekly-recap-<YYYY-MM-DD>.md` at the root of the current repo (today's date), then report the path of the created file. Without `--save`, write no file — print everything in the chat.
