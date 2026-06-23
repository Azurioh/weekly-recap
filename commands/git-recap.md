---
description: Recap a repository's changes over any time window you describe in plain language ("the last 2 days", "2 weeks", "1 month"). Detailed technical section + non-technical summary. Flags — --prs (GitHub PRs via gh), --save (markdown export), fr|en.
argument-hint: ["last 2 days" | 2w | "1 month"] [--prs] [--save] [fr|en]
allowed-tools: Bash(git rev-parse:*), Bash(git log:*), Bash(git shortlog:*), Bash(git diff:*), Bash(git branch:*), Bash(git remote:*), Bash(command -v gh:*), Bash(gh auth status:*), Bash(gh pr list:*), Bash(date:*), Write
---

Generate a recap of the repository's changes over a time window the user describes, in **two parts**: a detailed technical recap, then a non-technical summary. The goal is for a developer to get the useful detail, and for a non-technical reader (manager, client) to understand it without jargon.

## Arguments

Raw arguments: `$ARGUMENTS`

The arguments mix a **time window** (free text — the rest of the line) with optional flags:
- `--prs` or `--pr` → include GitHub Pull Requests. **Requires `gh` installed and authenticated.**
- `--save` or `--file` → also write the recap to a markdown file.
- `fr` / `en` → output language. Default: **the user's language in this conversation**, otherwise **French**.

### Interpreting the time window

Everything that isn't a flag describes **how far back to look**, in **plain language (French or English)** or shorthand. Interpret it into a concrete window. Examples you must handle:

- "les 2 derniers jours", "last 2 days", "2 jours", "2d" → 2 days
- "1 semaine", "last week", "1w" → 7 days
- "les 2 dernières semaines", "2 weeks", "2w" → 14 days
- "1 mois", "last month", "1mo" → 1 month
- "3 mois", "3mo" → 3 months
- "1 an", "1 year", "1y" → 1 year
- a bare number like "30" → 30 days
- **nothing at all** → default **7 days**

Resolve the phrase to a single **absolute start date** `START` (YYYY-MM-DD) so git and GitHub share the exact same boundary:
- macOS: `date -v-<N><U> +%F` where U is `d` (days), `w` (weeks), `m` (months), `y` (years) — e.g. `date -v-2w +%F`, `date -v-1m +%F`.
- Linux: `date -d "<N> <units> ago" +%F` — e.g. `date -d "2 weeks ago" +%F`.

If the phrase is ambiguous, pick the most reasonable reading. **Always state the resolved period** in the output (e.g. "Période : 14 derniers jours — depuis 2026-06-10") so the user can confirm the interpretation.

## 1. Collect the data (git)

First, verify you're inside a git repo: `git rev-parse --is-inside-work-tree`. If not, stop and say so clearly.

Using `START`, run these commands (adapt as needed):
- `git log --since="$START" --date=short --pretty=format:'%h%x09%ad%x09%an%x09%s'` — commit list (hash, date, author, subject).
- `git log --since="$START" --shortstat --pretty=format:'%h %s'` — change volume per commit (files, +/- lines) to aggregate.
- `git log --since="$START" --name-only --pretty=format:''` — touched files, to group by module/folder.
- `git shortlog -sn --since="$START"` — contributors and commit counts.
- `git branch --show-current` and `git remote get-url origin` — context (branch, repo).

**If there are no commits in the window**: say so clearly and don't fabricate anything. Suggest a longer window.

## 2. GitHub Pull Requests (only if `--prs`)

First check availability: `command -v gh` then `gh auth status`.
- If `gh` is missing or not authenticated: **flag it** ("install and log in to `gh` — `gh auth login` — to include PRs") and **continue without the PRs**, without blocking the rest.

Otherwise, using the same `START` date:
- `gh pr list --state merged --search "merged:>=$START" --json number,title,author,mergedAt,labels`
- `gh pr list --state open --json number,title,author,createdAt`

## 3. Output — two parts

Write in the chosen language. Start by stating the **resolved period**. **Don't just copy the commit list: group and interpret** the meaning of the changes from commit subjects and touched files.

### Part A — Detailed technical recap (for a developer)
- **Header**: repo, branch, resolved period (start date → today), number of commits, number of touched files, added/removed lines, contributors.
- **Grouped changes** by theme / module / feature. For each group: what changed and, where deducible, why.
- **Points of attention**: new modules or dependencies, structural refactors, breaking changes, schema/DB migrations, important fixes, technical debt introduced.
- If `--prs`: **merged PRs** (`#num — title — author`) and **still-open PRs** (work in progress).

### Part B — Non-technical summary (for a manager / client)
- 3 to 6 bullets in **plain language**, framed as **value and impact**: what was **added**, **improved**, **fixed**.
- **Zero jargon**: no file names, function names, or technical terms. A non-technical person must understand every bullet.

## 4. Export (only if `--save`)

Write both parts to `git-recap-<YYYY-MM-DD>.md` at the root of the current repo (today's date), then report the path of the created file. Without `--save`, write no file — print everything in the chat.
