---
name: vault-groom
argument-hint: "[today (default) | yesterday | YYYY-MM-DD]"
description: Grooming of the user's Obsidian vault. Processes a target day's notes — creates orphan files for referenced people/projects/companies, captures substantive meeting decisions into the Decisions folder, consolidates duplicates, updates MOCs, and flags strategic items. Defaults to TODAY; pass "yesterday" for the previous business day, or a YYYY-MM-DD date for a specific day. Use when asked to "groom the vault", run the daily/weekday vault cleanup, or process a day's Obsidian notes. Runs fully automatically without asking for approval.
---

You are grooming the user's Obsidian vault. Run fully automatically — create, consolidate, and update files without asking for approval — then report what you did in chat.

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Grep, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Step 1 — Determine the target day
Get today's date with a shell `date` command first, then resolve the target day (format YYYY-MM-DD) from the argument passed to the skill:

- **No argument (default) → today.** The target day is today's date.
- **`today` → today.** Same as the default.
- **`yesterday` → previous business day.** If today is Monday, the target day is the previous Friday; otherwise it is yesterday.
- **A specific date (`YYYY-MM-DD`) → that date.** Use it verbatim as the target day. If the argument isn't a valid date, stop and ask the user for a valid one.

State the resolved target day at the start of your run so it's clear which day is being groomed.

## Step 2 — Find what changed on the target day
1. Read the daily note at `6. Daily/<target-date>.md` if it exists — this is the primary log of what the user touched (it has "People mentioned", "Projects Touched", "Decisions / Signals", "Open loops" sections).
2. Also identify any `.md` files modified on the target day. Use Glob to list all markdown files (it sorts by modification time, newest first) and/or a `find` on the vault filtered by modification time. Ignore the `.obsidian/` folder.
3. Read the full content of every note touched on the target day.

## Vault conventions (follow these exactly for anything you create)
Folders: `1. People`, `2. Projects`, `3. Decisions`, `4. Companies`, `5. Meetings`, `6. Daily`, `7. Knowledge`.
Links use `[[Wikilink]]` style. Every note starts with YAML frontmatter including `type`, and `tags` (lowercase). People/company notes also have `status: active`.

Person note template (`1. People/<Name>.md`):
```
---
type: person
status: active
role:
tags:
  - people
---

## Role

## Context

## What they care about

## Active Links

## Communication Notes

## AI Recall Notes
```

Company note template (`4. Companies/<Company>.md`):
```
---
type: companies
status: active
industry:
tags:
  - companies
---
## Company Summary

## Why this company matters

## Relevant Links

## Current relationship

## AI Recall
```

Project note template (`2. Projects/<YYYY-MM-DD> - <Name>.md`): frontmatter `type: project`, `status: active`, `tags: [projects]`, then sections `## Summary`, `## Status`, `## Key Links`, `## Decisions`.

## Step 3 — Create orphan notes
Scan the target-day notes for mentions of people, projects, or companies that are referenced (as `[[X]]` links OR named in prose) but do NOT yet have their own file in the matching folder. For each genuine new entity, create a file using the correct template above, pre-filling anything you can confidently infer from the context (role, company, why it matters). Add a `[[wikilink]]` back-reference where natural. Do not create notes for trivial or ambiguous mentions.

## Step 4 — Capture decisions from the day's meetings
Turn substantive decisions made in the day's meetings into Decision notes. Read the `## Decisions` section of every meeting note dated the target day in `5. Meetings/`, plus any decisions logged in the daily note's `## Decisions / Signals` section. Keep only **substantive** decisions — choices with lasting effect (vendor/tooling/architecture/process/scope), not routine tasks, scheduling notes, or restatements of commitments. For each, glob `3. Decisions/` and skip any already covered by an existing note on the same subject (don't duplicate). Create the rest automatically (no approval needed) as `3. Decisions/<YYYY-MM-DD> - <Short title>.md` (date = the meeting date) using the Decision template at `0. Templates/3. Decision template.md`:
- Set `status: decided` for a decision that was made; `status: pending` if it was tabled/still open. Add lowercase topic tags after `decisions`.
- Fill **Decision** (what was decided), **Why** (rationale if stated), **Alternatives considered**, **Decision Owner** (`[[Name]]`), **Revisit Trigger**, and **AI Recall** — omit a section only when there's genuinely nothing for it; don't fabricate.
- Add a `- Decided in [[<YYYY-MM-DD> - <Meeting Title>]]` line so the decision traces back to its source meeting.
- Add the new note to `3. Decisions/Decisions MOC.md`.

## Step 5 — Consolidate duplicates
Look for duplicate or near-duplicate notes about the same person/project/company/topic (including loose top-level files like `Hawk proxy issue.md` that belong inside a numbered folder). Merge them: keep the best-located canonical file, fold in unique content from the others, update all `[[links]]` that pointed to the removed notes, and delete the redundant files. Preserve all substantive information.

## Step 6 — Update MOCs (Maps of Content)
The vault has no MOC index notes yet. Maintain one index note per relevant category at the folder root, named `<N>. <Category>/<Category> MOC.md` (e.g. `1. People/People MOC.md`), with `type: moc` frontmatter and a linked list of the notes in that category grouped sensibly. Create the MOC if missing; otherwise add links for any new notes from this run. Only touch MOCs for categories that gained or changed notes.

## Step 7 — Flag strategic items for review
From the target-day content, surface anything strategic the user should review before today's work: open decisions, commitments/deadlines, blockers, risks, or high-signal items. These are for the chat report only — do not create task files for them.

## Step 8 — Commit and push
After grooming is complete, commit and push all changes so the vault stays synced:
1. Append a one-line summary of this grooming run to `log.md` under today's `## YYYY-MM-DD` heading (newest first) — e.g. what was created/consolidated. Create the date heading if it doesn't exist.
2. Stage everything (`git add -A`), commit to `main` with a message like `chore: vault groom <target-date>`, and push to `origin main`. Do not create a branch — this is a personal sync repo.
3. If the push fails, report the error in the chat output; do not leave the run silently unpushed.

## Output (report in chat)
Report concisely in this structure:
- **Target day processed:** <date> (and how many notes were touched)
- **Orphan notes created:** list each new file with a one-line reason
- **Decision notes created:** each new `3. Decisions/` note and the meeting it came from (or "none")
- **Consolidations:** each merge (what folded into what, what was deleted)
- **MOCs updated:** which index notes changed
- **⚠️ Flagged for your review:** the strategic items, most important first
- **Committed & pushed:** the commit summary and push result (or the error if it failed)
If a section had no activity, say "none" for it. Keep it scannable.
