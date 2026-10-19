# Task: distill this conversation into devnotes

Turn what I learned in the conversation above into a learning note for my public `devnotes`
repo — or several linked notes if the topic is long. The repository rules further below are
the source of truth; follow them exactly.

Steps:

1. Name the single main question this conversation answered. If it covered two unrelated
   topics, ask me which one to write first and stop.
2. Pick the type, folder and file name using the rules. Prefer a folder that already exists.
   If a note on the same topic is already listed under "Current notes", stop and tell me its
   path — I will paste that note so you can update it instead.
3. If the note would pass ~1,500 words or answers several standalone questions, split it as the
   rules describe (one sub-question per note, linked under `## Related`).
4. Fill in the matching template. English only, concise, precise, easy to understand.
   Keep only the final, correct understanding; drop the back-and-forth.
   If the conversation used real work data, schemas, logs or names, rewrite the example as a
   neutral task with an invented schema and fake data (rules §5, *Public repo*).
5. Check every note against the checklist in the rules and fix anything that fails.

Output format — exactly this, with nothing before or after it:

- One code block per note, fenced with **four** backticks and the language `markdown`.
- Inside each block, line 1 is `<!-- path: folder/file-name.md -->`, then the note, starting with
  its front matter.
- Use `{{TODAY}}` for both `created` and `updated`.

I will copy everything and run `_meta/devnote.sh save`, which splits on the path lines.
