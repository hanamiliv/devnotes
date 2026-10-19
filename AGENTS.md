# AGENTS.md — rules for writing devnotes

This repo holds short learning notes. Most notes are distilled from conversations with an AI
assistant. These rules are the single source of truth for **any** agent or person adding a note:
Claude, Codex, Gemini, Cursor, ChatGPT, or a human with a text editor.

Goal of every note: **future-me, six months from now, gets the answer from the TL;DR in 10 seconds
and the full picture in one sitting.**

---

## 1. Workflow

1. Run `_meta/devnote.sh tree` (or list the folders) to see what already exists.
2. Choose the **type** (§2), the **folder** (§3) and the **file name** (§4).
3. If a note on the same topic exists, **update it** instead of creating a new one.
4. Copy the matching template from `_meta/templates/` and fill it in using the style rules (§5).
5. Run the checklist (§7).
6. Run `_meta/devnote.sh publish` to rebuild the index, commit and push.
   Without the script: update the index in `README.md` by hand and commit with the message format in §6.

---

## 2. Note types

| Type | Use when the note answers… | Template |
|---|---|---|
| `concept` | "How does X work?" / "Why does X behave like this?" | `_meta/templates/concept.md` |
| `howto` | "How do I do X?" — ordered steps | `_meta/templates/howto.md` |
| `comparison` | "X or Y? When do I pick which?" | `_meta/templates/comparison.md` |

If one conversation covers two unrelated topics, write two notes.

---

## 3. Choosing the folder

The folder is the **main technology** of the note: the word you would type first when searching
for it later.

1. Use a folder from the table below. Use the exact name; never create a synonym.
2. If a note is about how one tool *uses* another, pick the tool whose behaviour the note explains.
   "Kafka consumer offsets, shown with Go code" → `kafka`. "Goroutine leaks in a Kafka consumer" → `go`.
3. If the idea is not tied to one tool, use a domain folder (`databases`, `system-design`, …).
4. If nothing fits, create a new folder (lowercase, kebab-case, the product's common short name)
   **and add a row to this table in the same commit**.
5. One level only: `folder/note.md`. No sub-folders.

| Folder | Covers | Do not create |
|---|---|---|
| `go` | Go language, stdlib, tooling, concurrency, testing | `golang` |
| `postgres` | PostgreSQL: SQL, indexes, partitioning, MVCC, ops | `postgresql`, `pg` |
| `kafka` | Kafka brokers, topics, consumers, producers, Connect | `apache-kafka` |
| `clickhouse` | ClickHouse engines, queries, CDC into ClickHouse | `ch` |
| `databases` | Ideas shared by many databases: isolation levels, B-trees, CDC theory | `db` |
| `system-design` | Architecture patterns, idempotency, queues, caching, scaling | `architecture` |
| `git` | Git itself: commits, branches, rebase, history | |
| `github` | GitHub the platform: SSH keys, Actions, PRs, settings | `gh` |
| `docker` | Containers, images, Compose | `containers` |
| `macos` | Mac setup, terminal, shell, Homebrew | `mac`, `osx` |
| `ai-engineering` | LLMs, prompting, agents, skills, AI tools | `ai`, `llm`, `claude` |

---

## 4. File names

- Format: `folder/topic-in-kebab-case.md`, 2–6 words, lowercase ASCII.
- Name the **topic**, not the question: `postgres/partition-indexes.md`,
  not `postgres/how-do-indexes-work-on-partitioned-tables.md`.
- Do not repeat the folder name (`postgres/postgres-vacuum.md` ✗ → `postgres/vacuum.md` ✓).
- No dates, no numbers, no `how-to-` prefix (the type lives in front matter).
- Never rename an existing file; links may point to it.

---

## 5. Writing style

Language: **English only**, even when the conversation was in another language.

**Concise**
- Lead with the answer. The `TL;DR` alone must be useful.
- Length follows the content; long is fine when every paragraph earns its place.
- **Split** when a note passes ~1,500 words (body, excluding code) or answers more than one
  question that could stand alone. See *Splitting a long topic* below.
- One idea per bullet. A bullet is at most two lines.
- Cut filler: "In this note…", "It is important to note that…", "Basically", "Simply".

**Precise**
- Use exact names, flags, defaults, limits and versions ("PostgreSQL 16", "Go 1.23").
- Prefer a number to an adjective: "rewrites the whole table", not "is slow".
- If something is not verified, write `(unverified)` after it.

**Easy to understand**
- Plain words. Sentences under ~20 words. Active voice.
- Define a term the first time you use it, in one short clause.
- One small, runnable example beats three paragraphs. Show the expected output.
- Use a table when comparing two or more things on the same properties.

**Distil, don't transcribe**
- Keep the final, correct understanding. Drop the back-and-forth and the wrong turns.
- A wrong turn that is a common mistake goes into `Gotchas`.
- No traces of the chat: no "as discussed", "you asked", "the AI said", "great question".

**Splitting a long topic**
- Split by sub-question, not by size. Each part makes sense alone and has its own `TL;DR`.
- Name parts by subtopic, in the same folder: `postgres/partitioning.md`,
  `postgres/partition-indexes.md`, `postgres/partition-pruning.md`. No `part-1` numbering.
- Link the parts to each other in a `## Related` section.
- With three or more parts, the broadest note is the overview: its `Related` section lists the
  other parts in reading order.
- Publish all parts in one commit.

**Public repo — keep work private**

Many notes come from real work: a query on real data, a bug in a real service. The note keeps the
**lesson** and throws away **everything that came from the real system**. Rewrite it as a neutral
example task:

1. **Name the lesson** in one sentence, with no business words.
   "Get the latest row per group with `DISTINCT ON`", not "find each member's last tier change".
2. **Move it to a neutral domain** that has the same shape: same relationships (1-to-many,
   many-to-many), same data types, same edge cases (NULLs, duplicates, ties, time zones).
   Good defaults: online shop (`customers`, `orders`, `products`), blog (`authors`, `posts`),
   library (`books`, `loans`), sensors (`devices`, `readings`).
   Do not just rename the real tables. Pick a different domain.
3. **Invent the schema.** New table and column names, and only the columns the lesson needs
   (usually 2–6). Keep the types, keys and indexes that matter to the lesson.
4. **Invent the data.** 3–10 rows of obviously fake values: IDs `1, 2, 3`; names `Alice`, `Bob`;
   emails `@example.com`; round dates like `2026-01-01`; round amounts like `100.00`.
   Shape the rows so they still trigger the behaviour (the duplicate, the NULL, the tie).
5. **Recompute the results.** Expected output must match the fake data. Never paste real output.
6. **Round production numbers.** Size and speed may stay, rounded and without business meaning:
   "~50M rows", "from ~12 s to ~80 ms". No revenue, user counts or other business metrics.
7. **Strip the leftovers** in code comments, error messages, logs, stack traces, file paths,
   hostnames, URLs, config keys and commit hashes.
8. **No names.** No employer, client, product, team or internal service names, no ticket IDs,
   and no business domain that points to the employer.

Example:

| Learned at work | In the note |
|---|---|
| `acct_mvmt(cif_no, br_cd, amt_idr, posted_at)`, 40M real rows, "latest movement per account" | `orders(customer_id, store_id, amount, created_at)`, 6 fake rows, "latest order per customer" |
| Real output with real account numbers | Output recomputed from the 6 fake rows |
| "Query took 9.8 s on prod-db-03" | "~10 s on a ~40M-row table" |

Guards:
- `devnote lint` warns about patterns that look real: emails outside `example.com`, phone
  numbers, public IP addresses, UUIDs, internal hostnames.
- Optional: list private words (one per line) in `~/.config/devnote/denylist`, outside this
  repo. `devnote lint` then fails on any note that contains one. Never commit that file.

---

## 6. Format details

Front matter (required, first lines of the file):

```yaml
---
type: concept            # concept | howto | comparison
tags: [postgres, partitioning, indexing]   # 2–5 tags, lowercase, folder name first
created: 2026-10-19
updated: 2026-10-19      # change on every edit
---
```

Then exactly one `# Title` heading in Title Case, followed by the template sections in order.
Sections marked *(optional)* in a template may be deleted. Do not add new top-level sections.
Delete every `<!-- … -->` guidance comment from the template.

Sources: official documentation first, then source code, then good articles. Only link pages
that were actually used. Never invent a URL. If there are no sources, write `- None (from experience)`.

Commit message: `docs(<folder>): add <file-slug>` or `docs(<folder>): update <file-slug>`.

---

## 7. Checklist before publishing

- [ ] Folder is in the table in §3 (or the table was extended in this commit).
- [ ] No existing note already covers this topic.
- [ ] `TL;DR` answers the main question in 1–3 sentences.
- [ ] English only; no traces of the chat.
- [ ] No private work information: neutral domain, invented schema, fake data, recomputed
      output (§5, *Public repo*).
- [ ] Code and commands are complete and runnable as written; versions stated.
- [ ] Under ~1,500 words, or split into linked notes (§5).
- [ ] Every source link is real and was used.
