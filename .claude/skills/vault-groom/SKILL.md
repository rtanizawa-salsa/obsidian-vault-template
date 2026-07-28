---
name: vault-groom
argument-hint: "[today (default) | yesterday | YYYY-MM-DD]"
description: Grooming of the user's Obsidian vault. Processes a target day's notes — creates orphan files for referenced people/projects/companies, captures substantive meeting decisions into the Decisions folder, distills durable reference facts into Knowledge candidates, consolidates duplicates, updates MOCs, and flags strategic items. Defaults to TODAY; pass "yesterday" for the previous business day, or a YYYY-MM-DD date for a specific day. Use when asked to "groom the vault", run the daily/weekday vault cleanup, or process a day's Obsidian notes. Runs fully automatically without asking for approval.
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

Every note you create comes from a template file in `0. Templates/`. **Read the matching template and use it verbatim** — same frontmatter keys, same sections, same order — then fill in what the context supports. Never retype a structure from memory; the template file is the source of truth.

| Note | Path | Template |
| --- | --- | --- |
| Person | `1. People/<Name>.md` | `0. Templates/1. Person template.md` |
| Project | `2. Projects/<YYYY-MM-DD> - <Name>.md` | `0. Templates/2. Project template.md` |
| Decision | `3. Decisions/<YYYY-MM-DD> - <Short title>.md` | `0. Templates/3. Decision template.md` |
| Company | `4. Companies/<Company>.md` | `0. Templates/4. Company template.md` |
| Meeting | `5. Meetings/<YYYY-MM-DD> - <Title>.md` | `0. Templates/5. Meeting template.md` |
| Daily | `6. Daily/<YYYY-MM-DD>.md` | `0. Templates/6. Daily notes template.md` |
| Knowledge | `7. Knowledge/<Subject>.md` | `0. Templates/7. Knowledge template.md` |
| MOC | `<N>. <Category>/<Category> MOC.md` | `0. Templates/8. MOC template.md` |

Add lowercase topic tags after the template's own `type` tag (e.g. `people` → `people`, `platform`).

## Step 3 — Create orphan notes
Scan the target-day notes for mentions of people, projects, or companies that are referenced (as `[[X]]` links OR named in prose) but do NOT yet have their own file in the matching folder. For each genuine new entity, read the template from the table above and create the file from it, pre-filling anything you can confidently infer from the context (role, team, company, why it matters) and leaving the rest as the template's own placeholders. Add a `[[wikilink]]` back-reference where natural. Do not create notes for trivial or ambiguous mentions.

## Step 4 — Capture decisions from the day's meetings
Turn substantive decisions made in the day's meetings into Decision notes. Read the `## Decisions` section of every meeting note dated the target day in `5. Meetings/`, plus any decisions logged in the daily note's `## Decisions / Signals` section. Keep only **substantive** decisions — choices with lasting effect (vendor/tooling/architecture/process/scope), not routine tasks, scheduling notes, or restatements of commitments. For each, glob `3. Decisions/` and skip any already covered by an existing note on the same subject (don't duplicate). Create the rest automatically (no approval needed) as `3. Decisions/<YYYY-MM-DD> - <Short title>.md` (date = the meeting date), reading `0. Templates/3. Decision template.md` and using it verbatim as the structure:
- Set `status: decided` for a decision that was made; `status: pending` if it was tabled/still open. Add lowercase topic tags after `decisions`.
- Fill **Decision** (what was decided), **Why** (rationale if stated), **Alternatives considered**, **Decision Owner** (`[[Name]]`), **Revisit Trigger**, and **AI Recall** — omit a section only when there's genuinely nothing for it; don't fabricate.
- Add a `- Decided in [[<YYYY-MM-DD> - <Meeting Title>]]` line so the decision traces back to its source meeting.
- Add the new note to `3. Decisions/Decisions MOC.md`.

## Step 5 — Distill knowledge candidates
This is the "consolidation into long-term memory" pass: promote durable, reusable facts out of the day's raw notes toward the `7. Knowledge/` reference layer. Because Knowledge notes are hand-curated and high-effort, **do not auto-create or auto-edit them** — propose candidates into an inbox for the user to promote.

Scan the target-day notes (meetings, the day's Decision notes, and the daily note) for **knowledge**, which is different from a decision or an event:
- **Knowledge** = a durable, reusable fact — how something works, a policy/rule, a canonical value or spec (cutoff time, limit, id format, party role), a definition. It stays true across days and is worth looking up later.
- Not knowledge: a point-in-time choice (that's a **Decision**, handled in Step 4), or what happened / who said what (that stays in the meeting note).
Strong signals a fact is knowledge: it's phrased as a general rule/policy/spec, it's a concrete durable number or identifier, or the same fact is referenced across multiple notes.

Then:
1. Glob and read `7. Knowledge/` (the existing reference notes) and the candidates inbox `7. Knowledge/Knowledge candidates.md`. **Dedupe:** skip any fact already covered by an existing Knowledge note or already listed in the inbox.
2. Classify each genuine new candidate:
   - **New note** — no existing Knowledge note covers the topic → propose a note title and `area`.
   - **Update** — an existing Knowledge note covers the topic but is missing this fact (e.g. a new step or edge case for `Release process`) → name the target note and what to add.
3. Append each to `7. Knowledge/Knowledge candidates.md` (create it if missing, using the frontmatter below) as an unchecked checkbox item with: the fact in 1–2 lines, the proposed target (**New note:** `<title>` / **Update:** `[[Existing note]]`), a `Source:` `[[wikilink]]` back to the note it came from, and a confidence (high/medium/low).
4. Also list these candidates in the chat report (Output section) so the user can promote them immediately.

Only surface high-signal, durable facts — a handful at most per day, not every stray statement. If nothing qualifies, do nothing and report "none". Never fabricate a fact to fill the inbox.

Candidates inbox frontmatter (only when creating the file):
```
---
type: knowledge
tags:
  - knowledge
  - inbox
---
# Knowledge candidates (inbox)

Durable facts vault-groom distilled from daily notes, pending promotion into `7. Knowledge/`. Review, then move each into a real Knowledge note and delete the line.
```

## Step 6 — Consolidate duplicates
Look for duplicate or near-duplicate notes about the same person/project/company/topic (including loose top-level files like `Hawk proxy issue.md` that belong inside a numbered folder). Merge them: keep the best-located canonical file, fold in unique content from the others, update all `[[links]]` that pointed to the removed notes, and delete the redundant files. Preserve all substantive information.

## Step 7 — Update MOCs (Maps of Content)
Maintain one index note per relevant category at the folder root, named `<N>. <Category>/<Category> MOC.md` (e.g. `1. People/People MOC.md`), holding a linked list of that category's notes grouped sensibly. Create a missing MOC from `0. Templates/8. MOC template.md` — read it and use it verbatim as the structure, adding lowercase category tags after `moc`; otherwise just add links for any new notes from this run. Only touch MOCs for categories that gained or changed notes.

## Step 8 — Flag strategic items for review
From the target-day content, surface anything strategic the user should review before today's work: open decisions, commitments/deadlines, blockers, risks, or high-signal items. These are for the chat report only — do not create task files for them.

## Step 9 — Commit and push
After grooming is complete, commit and push all changes so the vault stays synced:
1. Append a one-line summary of this grooming run to `log.md` under today's `## YYYY-MM-DD` heading (newest first) — e.g. what was created/consolidated. Create the date heading if it doesn't exist.
2. Stage everything (`git add -A`), commit to `main` with a message like `chore: vault groom <target-date>`, and push to `origin main`. Do not create a branch — this is a personal sync repo.
3. If the push fails, report the error in the chat output; do not leave the run silently unpushed.

## Output (report in chat)
Report concisely in this structure:
- **Target day processed:** <date> (and how many notes were touched)
- **Orphan notes created:** list each new file with a one-line reason
- **Decision notes created:** each new `3. Decisions/` note and the meeting it came from (or "none")
- **Knowledge candidates:** each durable fact added to the inbox — the fact, its proposed target (new note or `[[existing note]]` to update), and confidence (or "none")
- **Consolidations:** each merge (what folded into what, what was deleted)
- **MOCs updated:** which index notes changed
- **⚠️ Flagged for your review:** the strategic items, most important first
- **Committed & pushed:** the commit summary and push result (or the error if it failed)
If a section had no activity, say "none" for it. Keep it scannable.
