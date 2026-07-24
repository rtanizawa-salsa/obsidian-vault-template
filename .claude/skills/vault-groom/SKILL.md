---
name: vault-groom
argument-hint: "[today (default) | yesterday | YYYY-MM-DD]"
description: Groom the Obsidian vault. Processes a target day's notes — creates orphan files for referenced people/projects/companies, consolidates duplicates, updates MOCs, and flags strategic items. Defaults to TODAY; pass "yesterday" for the previous business day, or a YYYY-MM-DD date for a specific day. Use when asked to "groom the vault", run the daily/weekday vault cleanup, or process a day's Obsidian notes. Runs fully automatically without asking for approval.
---

You are grooming the Obsidian vault. Run fully automatically — create, consolidate, and update files without asking for approval — then report what you did in chat.

## Vault location
This skill operates on the Obsidian vault in the current project (the repository root). Use the file tools (Read, Write, Edit, Grep, Glob) with paths relative to the vault root. If those tools can't access the vault, the folder connection was lost — say so and stop, telling the user to reconnect the vault folder.

Follow the conventions in [[schema]] and the note templates in `0. Templates/` for anything you create — do not invent a different structure.

## Step 1 — Determine the target day
Get today's date with a shell `date` command first, then resolve the target day (format YYYY-MM-DD) from the argument passed to the skill:

- **No argument (default) → today.** The target day is today's date.
- **`today` → today.** Same as the default.
- **`yesterday` → previous business day.** If today is Monday, the target day is the previous Friday; otherwise it is yesterday.
- **A specific date (`YYYY-MM-DD`) → that date.** Use it verbatim as the target day. If the argument isn't a valid date, stop and ask for a valid one.

State the resolved target day at the start of your run so it's clear which day is being groomed.

## Step 2 — Find what changed on the target day
1. Read the daily note at `6. Daily/<target-date>.md` if it exists — this is the primary log of what was touched (Main objectives, Decisions / Signals, Open loops).
2. Also identify any `.md` files modified on the target day. Use Glob to list all markdown files (it sorts by modification time, newest first) and/or a `find` on the vault filtered by modification time. Ignore the `.obsidian/` folder.
3. Read the full content of every note touched on the target day.

## Vault conventions (follow these exactly for anything you create)
Folders: `1. People`, `2. Projects`, `3. Decisions`, `4. Companies`, `5. Meetings`, `6. Daily`, `7. Knowledge`. Links use `[[Wikilink]]` style. Every note starts with YAML frontmatter including `type` and lowercase `tags`. People/company notes also have `status: active`.

For the exact frontmatter fields and sections per note type, use the matching file in `0. Templates/` and the spec in [[schema]]. Do not embed a divergent template here — the templates are the source of truth.

## Step 3 — Create orphan notes
Scan the target-day notes for mentions of people, projects, or companies that are referenced (as `[[X]]` links OR named in prose) but do NOT yet have their own file in the matching folder. For each genuine new entity, create a file from the correct template in `0. Templates/`, pre-filling anything you can confidently infer from the context (role, company, why it matters). Add a `[[wikilink]]` back-reference where natural. Do not create notes for trivial or ambiguous mentions.

## Step 4 — Consolidate duplicates
Look for duplicate or near-duplicate notes about the same person/project/company/topic (including loose top-level files that belong inside a numbered folder). Merge them: keep the best-located canonical file, fold in unique content from the others, update all `[[links]]` that pointed to the removed notes, and delete the redundant files. Preserve all substantive information.

## Step 5 — Update MOCs (Maps of Content)
Each content folder has an index note named `<Category> MOC.md` at its root (e.g. `1. People/0. People MOC.md`) with `type: moc` frontmatter and a linked list of the notes in that category, grouped sensibly. Create the MOC if missing; otherwise add links for any new notes from this run. Only touch MOCs for categories that gained or changed notes.

## Step 6 — Flag strategic items for review
From the target-day content, surface anything strategic worth reviewing before today's work: open decisions, commitments/deadlines, blockers, risks, or high-signal items. These are for the chat report only — do not create task files for them.

## Step 7 — Commit and push (if the vault is a synced git repo)
After grooming is complete, sync the vault so it stays up to date:
1. Append a one-line summary of this grooming run to `log.md` under today's `## YYYY-MM-DD` heading (newest first) — e.g. what was created/consolidated. Create the date heading if it doesn't exist.
2. Stage everything (`git add -A`), commit with a message like `chore: vault groom <target-date>`, and push. For a personal sync repo, commit to `main` directly rather than creating a branch.
3. If the push fails, report the error in the chat output; do not leave the run silently unpushed.

If the vault is not a git repo, skip this step and just save the files.

## Output (report in chat)
Report concisely in this structure:
- **Target day processed:** <date> (and how many notes were touched)
- **Orphan notes created:** list each new file with a one-line reason
- **Consolidations:** each merge (what folded into what, what was deleted)
- **MOCs updated:** which index notes changed
- **⚠️ Flagged for review:** the strategic items, most important first
- **Committed & pushed:** the commit summary and push result (or the error if it failed)

If a section had no activity, say "none" for it. Keep it scannable.
