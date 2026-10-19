---
name: devnote
description: Distill what was learned in the current conversation into an English note in the user's devnotes repo (github.com/hanamiliv/devnotes), split into linked notes when long, file it in the right folder, and push. Use when the user says /devnote, "save this to devnotes", "summarize this into my notes", or "simpan ke devnotes".
---

# devnote

Turn the learning in this conversation into a note (or a few linked notes) in the devnotes repo.
The repo's `AGENTS.md` is the source of truth for folders, names, style and templates.
This skill only says how to reach the repo and in which order to work.

Talk to the user in their language. The note itself is always English.

## 1. Reach the repo — first option that works

1. **Already inside it**: the working directory contains `AGENTS.md` and `_meta/devnote.sh`.
2. **Clone with push access**: clone `https://github.com/hanamiliv/devnotes`. If the environment
   needs a repo to be attached before it can push (for example an `add_repo` tool), attach
   `hanamiliv/devnotes` with push access first and use the clone command it gives.
   Set the commit identity from the existing history:
   `git config user.name "$(git log -1 --format=%an)"` and the same with `%ae` for `user.email`.
3. **A folder on the user's computer** you can write to but cannot run `git commit` in: write the
   file there, then tell the user to run `dn publish` (alias for `_meta/devnote.sh publish`).
4. **No access at all**: produce the note as a downloadable `.md` file whose first line is
   `<!-- path: folder/file.md -->`, and tell the user to run `dn save <file>`.

## 2. Load the rules

Read all of `AGENTS.md`, then run `_meta/devnote.sh tree` (or list the folders) to see existing notes.
Open the template in `_meta/templates/` for the type you pick.

## 3. Distill

- Source: this conversation, or the part the user points to.
- Follow `AGENTS.md`: type, folder, file name, update-instead-of-duplicate, style, privacy.
- If an existing note covers the topic, read it and merge into it; keep `created`, bump `updated`.
- Long is fine. If the note would pass ~1,500 words or answers several standalone questions,
  split it as `AGENTS.md` §5 describes and publish all parts in one commit.
- If the conversation covered two unrelated topics, ask which to save (or save both as two notes if
  the user said so).
- The repo is public. Assume anything from the conversation may be real work material.
  Follow `AGENTS.md` §5 *Public repo*: keep the lesson; move the example to a neutral domain;
  invent the schema; use 3–10 rows of fake data; recompute the output; round production numbers;
  strip names, hostnames, logs and IDs, including names you know from memory or earlier context.
  Never copy a real table, column, row or query result into the note.
- Check version-specific facts you are not sure of (web search if available). If a runtime is
  available, run the example and paste the real output; otherwise keep it minimal.

## 4. Confirm once

Show the user, in one short block, for each note: path (new or update), title, the TL;DR line, word count.
Ask: **Push** / **Edit** / **Cancel** (use a multiple-choice question tool if one exists).
Skip this step when the user already said to push without asking.

## 5. Write and publish

Write the file(s), then run `_meta/devnote.sh publish`.
It lints the note, rebuilds the README index, commits (`docs(<folder>): add <slug>`) and pushes.
If lint reports an ERROR, fix the note and run it again; do not use `--force` unless the user asks.

Finish with one line per note: the file path and its GitHub link
`https://github.com/hanamiliv/devnotes/blob/main/<path>`.
