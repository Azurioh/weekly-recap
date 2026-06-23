---
description: Récap des changements des 7 derniers jours — section technique détaillée + résumé non-technique. Flags — --prs (inclut les PRs GitHub via gh), --save (export markdown), --days N, fr|en.
argument-hint: [--prs] [--save] [--days N] [fr|en]
allowed-tools: Bash(git rev-parse:*), Bash(git log:*), Bash(git shortlog:*), Bash(git diff:*), Bash(git branch:*), Bash(git remote:*), Bash(command -v gh:*), Bash(gh auth status:*), Bash(gh pr list:*), Bash(date:*), Write
---

Tu génères un récap des changements du dépôt sur une fenêtre récente (7 jours par défaut), en **deux parties** : un récap technique détaillé, puis un résumé non-technique. Le but est qu'un dev ait le détail utile, et qu'un non-technique (manager, client) comprenne sans jargon.

## Arguments

Arguments bruts : `$ARGUMENTS`

Parse les flags suivants (l'ordre n'importe pas) :
- `--prs` ou `--pr` → inclure les Pull Requests GitHub. **Nécessite `gh` installé et authentifié.**
- `--save` ou `--file` → écrire aussi le récap dans un fichier markdown.
- `--days N` → remplace la fenêtre de 7 jours par N jours.
- `fr` / `en` → langue de sortie. Par défaut : **la langue de l'utilisateur dans cette conversation**, sinon **français**.

## 1. Collecte des données (git)

D'abord, vérifie qu'on est dans un repo git : `git rev-parse --is-inside-work-tree`. Sinon, arrête-toi et signale-le clairement.

Définis la fenêtre `SINCE` : `7 days ago` par défaut (ou `N days ago` si `--days N`).

Lance ces commandes (adapte au besoin) :
- `git log --since="$SINCE" --date=short --pretty=format:'%h%x09%ad%x09%an%x09%s'` — liste des commits (hash, date, auteur, sujet).
- `git log --since="$SINCE" --shortstat --pretty=format:'%h %s'` — volume de changements par commit (fichiers, lignes +/-) à agréger.
- `git log --since="$SINCE" --name-only --pretty=format:''` — fichiers touchés, pour regrouper par module/dossier.
- `git shortlog -sn --since="$SINCE"` — contributeurs et nombre de commits.
- `git branch --show-current` et `git remote get-url origin` — contexte (branche, dépôt).

**Si aucun commit dans la fenêtre** : dis-le clairement et ne fabrique rien. Propose `--days N` pour élargir.

## 2. Pull Requests GitHub (uniquement si `--prs`)

Vérifie d'abord la dispo : `command -v gh` puis `gh auth status`.
- Si `gh` absent ou non authentifié : **signale-le** (« installe et connecte `gh` — `gh auth login` — pour inclure les PRs ») et **continue sans les PRs**, sans bloquer le reste.

Sinon, calcule la date de début (macOS : `date -v-7d +%F` ; Linux : `date -d '7 days ago' +%F` ; adapte si `--days N`), puis :
- `gh pr list --state merged --search "merged:>=<DATE>" --json number,title,author,mergedAt,labels`
- `gh pr list --state open --json number,title,author,createdAt`

## 3. Sortie — deux parties

Écris dans la langue choisie. **Ne recopie pas bêtement la liste des commits : regroupe et interprète** le sens des changements à partir des sujets de commits et des fichiers touchés.

### Partie A — Récap technique détaillé (pour un dev)
- **En-tête** : dépôt, branche, période réelle couverte (dates), nb de commits, nb de fichiers touchés, lignes ajoutées/supprimées, contributeurs.
- **Changements regroupés** par thème / module / feature. Pour chaque groupe : ce qui a changé et, si déductible, pourquoi.
- **Points d'attention** : nouveaux modules ou dépendances, refactors structurants, breaking changes, migrations de schéma/DB, corrections importantes, dette technique introduite.
- Si `--prs` : **PRs mergées** (`#num — titre — auteur`) et **PRs encore ouvertes** (travail en cours).

### Partie B — Résumé non-technique (pour un manager / client)
- 3 à 6 puces en **langage simple**, formulées en **valeur et impact** : ce qui a été **ajouté**, **amélioré**, **corrigé**.
- **Zéro jargon** : pas de noms de fichiers, de fonctions, ni de termes techniques. Une personne non-technique doit comprendre chaque puce.

## 4. Export (uniquement si `--save`)

Écris les deux parties dans `weekly-recap-<YYYY-MM-DD>.md` à la racine du repo courant (date du jour), puis indique le chemin du fichier créé. Sans `--save`, n'écris aucun fichier — affiche tout dans le chat.
