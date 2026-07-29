---
name: distill-source
argument-hint: "[filename in 9. Sources | URL | all (default)]"
description: Distill a raw source into durable Knowledge notes. Reads a source dropped in `9. Sources/` (PDF, markdown, text, transcript, exported article) or a URL, extracts the reusable facts, and writes clean single-subject notes into `7. Knowledge/` — merging into existing notes on the same topic, preserving provenance, and updating the Knowledge MOC. Runs fully automatically. Use when the user says "distill this source", "process the sources folder", "turn this into knowledge", "ingest <file>", or drops a file in `9. Sources/` and asks to process it.
---

You are distilling raw source material into the user's durable Knowledge base. This is the deliberate counterpart to `vault-groom`: the user has chosen a source worth keeping, and your job is to transform dense raw material into clean, deduplicated, well-structured reference notes — the way a good wiki page distills a primary source. Run fully automatically — read, distill, write, and merge without asking for approval — then report what you did in chat.

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Grep, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Step 1 — Resolve the source(s)
Resolve from the argument passed to the skill:

- **No argument, or `all` → every unprocessed file in `9. Sources/`.** Glob `9. Sources/` and process each file. **Skip `README.md`** and any file with `type: system` frontmatter — those are not sources.
- **A filename or path → that single file.** Resolve it inside `9. Sources/` first; if not found there, treat the argument as a path relative to the vault root.
- **A URL (`http://` / `https://`) → fetch it.** Use WebFetch to retrieve the page content. (WebFetch is a deferred tool — load it with ToolSearch `select:WebFetch` before calling.)

If nothing resolves (empty folder, missing file, unreachable URL), say so and stop.

## Step 2 — Read and extract the source
Read the full source content before distilling:
- **Markdown / text / transcript** — Read the whole file.
- **PDF** — Read with the `pages` parameter. For long PDFs, read in page ranges and cover the whole document; do not distill from only the first pages.
- **URL** — use the fetched content.

Extract faithfully. Everything you write must be traceable to the source — **never fabricate, infer beyond what the source supports, or pad with outside knowledge.** If the source is ambiguous on a point, say so in the note rather than resolving it silently.

## Step 3 — Identify the durable knowledge
Pull out what belongs in long-term reference, and drop the rest. Keep **knowledge**: how something works, a policy or rule, a canonical value or spec (cutoff time, limit, id format, code table, party role), a definition, a procedure. Drop ephemera: narrative filler, marketing, one-off context, anything that won't be true or useful later.

Follow the vault's **one-subject-per-note** rule. A dense source usually yields *several* single-subject notes, not one sprawling note — e.g. an engineering spec might become separate `API rate limits`, `Release process`, and `Glossary` notes. Split by topic, not by source.

## Step 4 — Dedupe and merge against existing Knowledge
Before writing anything, Glob and Read `7. Knowledge/` so you know what already exists. For each distilled topic:
- **Existing note covers it** → **enrich that note**, don't duplicate. Integrate the new facts into the right section (add a row, an FAQ, an edge case, a `## <New aspect>` section). **Preserve all existing hand-written content** — you are adding to a curated note, not rewriting it. If the source contradicts an existing fact, keep both and flag the conflict inline (e.g. `> ⚠️ Source X says 3 days; existing note said 5 — reconcile`) rather than silently overwriting.
- **No existing note** → create a new one (Step 5).

Also check the candidates inbox — locate it by Glob (`7. Knowledge/*Knowledge candidates*.md`) rather than a hard-coded path, since the filename may carry a sort prefix: if a fact you're now writing as a real note was sitting there as a candidate, remove that candidate line — it's been promoted.

## Step 5 — Write the Knowledge notes
Read `0. Templates/7. Knowledge template.md` and use it verbatim as each new note's structure — same frontmatter keys, same sections, same order. Create each note as `7. Knowledge/<Topic>.md` and fill it in as follows:
- `area` — e.g. Engineering, Process, Product. Leave blank if genuinely unclear.
- `tags` — keep `knowledge`, then add lowercase topic tags.
- `# <heading>` — the topic.
- **Summary** — 2–4 lines: what this topic is and why it matters.
- **Details** — the clean, structured distillation: sub-headings per aspect, tables for canonical values, blockquotes for verbatim rules. Faithful to the source; deduplicated; organized for lookup, not narrative.
- **Source** — first bullet is the source title / document name, then `Type:` (PDF | article | transcript | spec | webpage), `Origin:` (author / publisher / URL if any), and `Processed:` (`YYYY-MM-DD` — get today's date with a shell `date` command).

Provenance rules for `## Source`:
- Capture enough that the knowledge stays traceable **after the raw source file is deleted** from `9. Sources/`.
- For a URL, record the full link. For a file, record its original filename and any title/author inside it.
- **Do not** write a `[[wikilink]]` to the source file — it lives in a transient folder and will be removed, leaving a broken link. Use plain text / a plain markdown link instead.

Cross-link into the rest of the vault where natural: `[[Company]]`, `[[Person]]`, or related `[[Knowledge note]]` wikilinks (these targets are permanent, unlike the source).

## Step 6 — Update the Knowledge MOC
Add every newly created note to `7. Knowledge/0. Knowledge MOC.md` as a `[[wikilink]]` list item. Notes you only enriched are already listed — don't re-add them.

## Step 7 — Leave the source in place
Do **not** delete anything from `9. Sources/` — the user cleans up that folder themselves once they've reviewed the result. In your report, list exactly which source files were processed so the user knows what is now safe to delete.

## Step 8 — Log the run
Append a one-line summary to `log.md` under today's `## YYYY-MM-DD` heading (newest first) — e.g. `distilled <source> → <notes created/enriched>`. Create the date heading if it doesn't exist. Do **not** commit or push — leave the vault's git state for the user or the next `vault-groom` run to sync, unless the user explicitly asks you to commit.

## Output (report in chat)
Report concisely:
- **Source(s) processed:** each file/URL, with a one-line note of what it was
- **Knowledge notes created:** each new `7. Knowledge/` note and a one-line summary of what it captures
- **Knowledge notes enriched:** each existing note you added to, and what you added
- **Conflicts flagged:** any place the source contradicted an existing note (or "none")
- **MOC updated:** yes/no
- **🧹 Safe to clean up:** the source files in `9. Sources/` you finished distilling, so the user can delete them
If a section had no activity, say "none" for it. Keep it scannable.
