# git-recap

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin that recaps the changes in your repository over **any time window you describe in plain language** — a **detailed technical section** for developers, plus a **plain-language summary** for non-technical readers (managers, clients).

## Install

From the [`azurioh-plugins`](https://github.com/Azurioh/claude-plugins) marketplace:

```
/plugin marketplace add Azurioh/claude-plugins
/plugin install git-recap@azurioh-plugins
```

## Usage

Run inside any git repository. Describe the window in your own words (French or English) — or pass nothing for the last 7 days:

```
/git-recap                       # last 7 days (default)
/git-recap les 2 derniers jours  # last 2 days
/git-recap 2 weeks               # last 2 weeks
/git-recap 1 mois --prs          # last month, including GitHub PRs
/git-recap 1y --save             # last year, also exported to markdown
```

The window accepts plain phrases ("les 2 dernières semaines", "last 2 days", "1 month") and shorthand (`2d`, `2w`, `1mo`, `1y`, or a bare number = days). The command always reports the period it resolved so you can confirm the interpretation.

### Flags

| Flag | Effect |
|---|---|
| `--prs` (or `--pr`) | Also include GitHub Pull Requests (merged & open). **Requires `gh` installed and authenticated** (`gh auth login`). |
| `--save` (or `--file`) | Also write the recap to `git-recap-<YYYY-MM-DD>.md` at the repo root. |
| `fr` / `en` | Output language. Defaults to your conversation language, otherwise French. |

## Output

- **Part A — Technical recap:** repo/branch/period header, commit & change volume, contributors, changes grouped by module/feature, and points of attention (refactors, breaking changes, migrations, new deps). With `--prs`: merged and open PRs.
- **Part B — Non-technical summary:** 3–6 plain-language bullets framed as value and impact (added / improved / fixed), no jargon.

By default everything is printed in the chat; nothing is written to disk unless you pass `--save`.
