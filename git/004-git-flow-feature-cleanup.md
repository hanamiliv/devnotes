---
type: howto
tags: [git, git-flow, branching, pull-requests]
created: 2026-11-03
updated: 2026-11-03
---

# Git Flow Feature Branches With a PR Workflow

> **TL;DR** — Use `git flow feature start` (and optionally `publish`), but skip `finish` when merges happen through pull requests. After the PR is merged, update `develop`, delete the local branch, and run `git fetch --prune`. If `git branch -d` says "not fully merged", the PR was squash- or rebase-merged: confirm it is merged, then use `-D`.

## Before You Start

- `git flow init` has run once in the repo (defaults: `develop`, `feature/` prefix).
- Tested with Git 2.43.

## Steps

### 1. Start and share the feature

`start` branches from `develop`; `publish` pushes it so you can open a PR.

```bash
git flow feature start add-search     # creates and checks out feature/add-search
git flow feature publish add-search   # pushes to origin and sets upstream
```

### 2. Merge through the PR, not `finish`

`git flow feature finish` merges into `develop` **locally** and deletes the branch. With a PR
workflow the merge already happens on the server, so `finish` is redundant and can create a
second, divergent merge.

### 3. Update develop after the PR is merged

```bash
git checkout develop
git pull origin develop
```

### 4. Delete the local branch and prune remote refs

```bash
git branch -d feature/add-search
git fetch --prune          # removes stale origin/feature/add-search
```

If the server did not auto-delete the remote branch:

```bash
git push origin --delete feature/add-search
```

### 5. Handle "not fully merged" after a squash merge

A squash or rebase merge creates **new** commits on `develop`. `git branch -d` compares commit
hashes, so it thinks the branch is unmerged.

```text
$ git branch -d feature/add-search
error: the branch 'feature/add-search' is not fully merged.
```

Confirm the PR shows "Merged", then force-delete:

```bash
git branch -D feature/add-search
```

### 6. Start follow-up work from fresh develop

Never reuse a merged feature branch. It carries already-merged history into the next PR and
causes duplicate commits or confusing conflicts.

```bash
git checkout develop && git pull
git flow feature start add-search-followup
```

## Verify

```bash
git branch --list 'feature/*'       # merged features no longer listed
git branch -r --list 'origin/feature/add-search'   # empty
```

## Gotchas

- `git branch --merged develop` misses squash-merged branches; it only finds real merge commits.
- After `git fetch --prune`, a branch whose remote was deleted shows `[origin/...: gone]` in
  `git branch -vv`. That is a good list of candidates to check and delete with `-D`.
- `git branch -d` checks against the branch's upstream (or `HEAD`). Pull `develop` first, or it
  may refuse even for a normal merge.
- `-D` deletes unmerged work without asking. Check the PR status before using it.

## Quick Reference

| Task | Command |
|---|---|
| Start feature | `git flow feature start <name>` |
| Push feature for a PR | `git flow feature publish <name>` |
| Delete merged branch | `git branch -d feature/<name>` |
| Delete squash-merged branch | `git branch -D feature/<name>` |
| Drop stale remote refs | `git fetch --prune` |
| Bulk-delete merge-committed features | `git branch --merged develop \| grep 'feature/' \| xargs git branch -d` |
| List branches whose remote is gone | `git branch -vv \| grep ': gone]'` |

## Sources

- [git-branch documentation](https://git-scm.com/docs/git-branch)
- [git-fetch documentation](https://git-scm.com/docs/git-fetch)
- [A successful Git branching model (git-flow)](https://nvie.com/posts/a-successful-git-branching-model/)
