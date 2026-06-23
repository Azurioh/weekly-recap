# weekly-recap

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin that gives you a recap of the changes in your repository over the last 7 days — a **detailed technical section** for developers, plus a **plain-language summary** for non-technical readers (managers, clients).

## Install

From the [`azurioh-plugins`](https://github.com/Azurioh/claude-plugins) marketplace:

```
/plugin marketplace add Azurioh/claude-plugins
/plugin install weekly-recap@azurioh-plugins
```

## Usage

Run inside any git repository:

```
/weekly-recap
```

### Flags

| Flag | Effect |
|---|---|
| `--prs` (or `--pr`) | Also include GitHub Pull Requests (merged & open). **Requires `gh` installed and authenticated** (`gh auth login`). |
| `--save` (or `--file`) | Also write the recap to `weekly-recap-<YYYY-MM-DD>.md` at the repo root. |
| `--days N` | Use an N-day window instead of 7. |
| `fr` / `en` | Output language. Defaults to your conversation language, otherwise French. |

Examples:

```
/weekly-recap --prs
/weekly-recap --days 14 --save
/weekly-recap en --prs --save
```

## Output

- **Part A — Technical recap:** repo/branch/period header, commit & change volume, contributors, changes grouped by module/feature, and points of attention (refactors, breaking changes, migrations, new deps). With `--prs`: merged and open PRs.
- **Part B — Non-technical summary:** 3–6 plain-language bullets framed as value and impact (added / improved / fixed), no jargon.

By default everything is printed in the chat; nothing is written to disk unless you pass `--save`.
