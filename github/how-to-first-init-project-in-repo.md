# Your First Commit Into a Repo That Already Has One

You have a folder on your laptop with working code in it. You create a new repository on the
server's web UI, tick nothing in particular, and it hands you a page of instructions. You run
`git push`, and git refuses.

The instructions on that page end with `git push -uf origin main`. That `-f` is doing a lot of
work, and almost nobody reads it before pasting.

This walks through the same situation without a force push: a linear history, the remote's
existing commit preserved, your commit author set to the right identity, and — if you also want
the project on a second remote — a clean history over there without dragging the old one along.

Every command here was run end to end against a throwaway remote before being written down.
Git 2.50.

---

## 1. Look before you touch anything

Four read-only commands. Each answers one question, and none of them change a thing.

```bash
# Does the remote already have commits? This decides everything that follows.
git ls-remote git@git.example.com:you/my-project.git

# Is SSH even working? Ask before blaming git.
ssh -T git@git.example.com

# What would actually get committed?
find . -not -path './.git/*' -type f | sort

# Anything in here that should not leave the machine?
grep -rniE "(bearer [A-Za-z0-9_.-]{20,}|secret|api[_-]?key|BEGIN [A-Z ]*PRIVATE KEY)" .
```

`git ls-remote` is the one that matters most. It lists the remote's refs without cloning
anything, so you learn in one second whether you are pushing into an empty repo or onto somebody
else's commit:

```
0a5abc13f203e95c0dbdfd20212c6087a0aa5260	HEAD
0a5abc13f203e95c0dbdfd20212c6087a0aa5260	refs/heads/main
```

A `refs/heads/main` line means there is already a commit there. If the output is empty, the repo
is bare and the rest of this article is unnecessary — just push.

The `find` and `grep` are worth the ten seconds because **a push publishes**. Getting a commit
back off a remote is far more expensive than looking at it first, and `.DS_Store`,
`node_modules/`, and a stray `dump.sql` all travel quietly if nobody checks.

---

## 2. Initialise with the branch name you actually want

```bash
git init -b main
```

If `init.defaultBranch` is not configured, older git creates `master` while your remote is on
`main`, and now you are juggling two branch names for no reason. `-b` needs git 2.28 or newer; on
anything older, `git init` followed by `git branch -m main`.

---

## 3. Set the identity on the repo, not globally

```bash
git config user.name  "Your Name"
git config user.email "you@company.example"
```

Note the absence of `--global`. Two reasons this matters:

- A commit authored with an email that is not registered on your server account **will not be
  linked to your profile**, and some instances have push rules that reject it outright.
- Setting it globally changes every other repository on the machine, including the personal ones.

Confirm the two are separate:

```bash
git config --get user.email           # you@company.example
git config --global --get user.email  # whatever it was before, untouched
```

---

## 4. What a plain push actually says

Add the remote, make your commit, and try it, just to see the failure:

```bash
git remote add origin git@git.example.com:you/my-project.git
git add -A && git commit -m "wip"
git push origin main
```

```
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'git@git.example.com:you/my-project.git'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally. This is usually caused by another repository pushing to
hint: the same ref. If you want to integrate the remote changes, use
hint: 'git pull' before pushing again.
```

Your history and the remote's history share no commits at all, so git cannot move the branch
forward without throwing one of them away. It declines to guess which.

`git push -uf` makes the error disappear by throwing the remote's commit away. Sometimes that is
genuinely fine — see the last section. But if that commit is not yours, or if anyone has already
cloned the repo, you have just deleted work you did not own and broken their clone. It is worth
thirty more seconds to not find out which case you were in.

---

## 5. Fetch the remote's history, then read it

```bash
git fetch origin main
git show --stat --format='%H %s' FETCH_HEAD
git ls-tree -r --name-only FETCH_HEAD
```

```
0a5abc13f203e95c0dbdfd20212c6087a0aa5260 Initial commit

 .gitlab-ci.yml | 1 +
 README.md      | 3 +++
```

Those last two commands are not ceremony. Nine times out of ten the commit holds nothing but a
generated `README.md` you are happy to replace. The tenth time it also holds a CI config, a
license you picked from a template dropdown, or a `.gitignore` — things you want to **keep**.

---

## 6. Put your work on top of that commit

```bash
git reset --soft FETCH_HEAD
git add -A
git status --short
```

`reset --soft` is the piece that feels wrong the first time, so here is exactly what it does: it
points `HEAD` at the remote's commit and **leaves your working tree and index alone**. Nothing on
disk is touched. Your files are still there; git has simply changed its mind about what they are
being compared against.

Which is why the status output is the important part:

```
D  .gitlab-ci.yml
M  README.md
A  index.js
A  src/a.txt
```

Read that `D`. `.gitlab-ci.yml` exists in the remote's commit and not in your folder, so
`git add -A` staged its **deletion**. Commit now and you silently remove it.

If you want to keep it, take it from the fetched commit before committing:

```bash
git checkout FETCH_HEAD -- .gitlab-ci.yml
git status --short
```

```
M  README.md
A  index.js
A  src/a.txt
```

The `D` is gone. Now commit and push:

```bash
git commit -m "feat: import my-project"
git push -u origin main
```

```
* 6f47633 feat: import my-project
* 0a5abc1 Initial commit
```

Your commit's parent is the remote's commit, which makes the push a **fast-forward**. Linear
history, no merge commit, no `--force`, and the original commit still in place.

---

## 7. Mirroring to a second remote, with a clean history

A different problem, often arriving right after the first: the same project should also live on a
personal account, but its history carries things you would rather not bring — placeholder commits,
old sample values, internal names from an earlier revision.

Sanitising the files and committing on top **does not help**. `git log -p` still shows every old
value to anyone who clones.

An orphan branch does:

```bash
git checkout --orphan clean-main
git add -A
git commit -m "feat: initial import"
git checkout main
```

`--orphan` creates a branch with **no parent**. The tree is identical to where you branched from;
the history is simply absent. Prove both:

```bash
git rev-list --parents -n 1 clean-main   # one sha, no parent sha after it
git diff --stat main clean-main          # empty: the contents match exactly
```

Then publish that branch as `main` on the second remote:

```bash
git remote add personal git@github.com:you/my-project.git
git push personal clean-main:main
```

> **`git switch --orphan` is not the same command.** Verified on git 2.50: `git checkout --orphan`
> keeps your working tree and leaves everything staged, while `git switch --orphan` gives you an
> **empty** working tree. Both are documented behaviour; only one of them is what you want here.

One honest limit: this gives the *second* remote a clean history. On the first remote the old
commits are still there, because you chose not to rewrite a branch that had already been pushed.
For a repo other people may have cloned, that is the right trade.

---

## 8. The trap: clean files, dirty history

This is the part that nearly got missed, and it is the reason the article exists.

Every file is clean:

```bash
grep -ril "company" . --exclude-dir=.git
# no output
```

The history is not:

```bash
git log -p clean-main | grep -i "company"
```

```
Author: Your Name <you@company.example>
```

The commit **object** stores the identity, and `git log -p` prints it. A personal repository with
a work email stamped on every commit is not what you meant to publish — and grepping the files
would never have told you.

Author and committer are two separate fields, and both are stored:

```bash
git log -1 --format='author:    %an <%ae>%ncommitter: %cn <%ce>' clean-main
```

```
author:    Your Name <you@company.example>
committer: Your Name <you@company.example>
```

While the branch is still unpushed, fixing it is free:

```bash
git checkout clean-main
GIT_AUTHOR_NAME="Your Name"     GIT_AUTHOR_EMAIL="you@personal.example" \
GIT_COMMITTER_NAME="Your Name"  GIT_COMMITTER_EMAIL="you@personal.example" \
  git commit --amend --no-edit --reset-author
git checkout main
```

```
author:    Your Name <you@personal.example>
committer: Your Name <you@personal.example>
```

The environment variables set both fields; `--reset-author` makes sure the author moves too and
not just the committer. Your main branch keeps the work identity it needs.

The general lesson is worth more than the command: **audit the history, not just the working
tree.** A grep over files only ever describes the present.

---

## 9. Verify before you walk away

| Check | Command | Expect |
|---|---|---|
| Push landed | `git ls-remote origin main` | same sha as `git rev-parse HEAD` |
| History is linear | `git log --oneline --graph` | your commit on top of theirs, no merge |
| Nothing was deleted by accident | `git show --stat HEAD` | no file you meant to keep listed as removed |
| Orphan really is orphaned | `git rev-list --parents -n 1 clean-main` | one sha, no parent |
| Mirror contents match | `git diff main clean-main` | empty |
| Identity per branch | `git log -1 --format='%an <%ae>' <branch>` | the address you intended, per branch |
| Global config untouched | `git config --global --get user.email` | unchanged |
| It works from a clone | clone to a temp dir and open/run it | behaves the same as your folder |

That last one earns its place. Everything else checks your folder; a fresh clone checks what you
actually published.

---

## 10. When `--force` is the right answer

This is not a rule against force pushes. If the commit sitting on the remote is one **you** made
through the web UI four minutes ago, and nobody has cloned the repo, rewriting it is completely
reasonable and a good deal shorter.

Use the safer spelling:

```bash
git push --force-with-lease origin main
```

`--force-with-lease` refuses to run if the remote moved since your last fetch, which is exactly
the case where a plain `--force` quietly destroys something. There is very little reason to type
`--force` when `--force-with-lease` exists.

---

## The short version

```bash
git ls-remote <url>                    # is anything there?
git init -b main
git config user.email "you@company.example"
git remote add origin <url>
git fetch origin main
git ls-tree -r --name-only FETCH_HEAD  # what is in their commit?
git reset --soft FETCH_HEAD
git add -A
git status --short                     # read every D line
git commit -m "feat: import my-project"
git push -u origin main                # fast-forward
```

Three habits generalise past this particular mess: check the remote before you push to it, read
`git status` before you commit, and remember that history stores identities as well as files.
