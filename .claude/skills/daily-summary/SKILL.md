---
name: daily-summary
argument-hint: "[today (default) | YYYY-MM-DD]"
description: Interview the user to fill in the current day's Obsidian daily note. Pulls the day's Linear activity (issues done/started/created/updated and comments made) for confirmation, checks Granola for meetings on that date missing from the Meetings folder, records substantive decisions from those meetings into the Decisions folder, then asks, one topic at a time, about tickets/tasks done, blocking issues found during the day, and open loops still remaining, and writes the answers into `6. Daily/<date>.md` using the vault's Daily template — preserving anything already there. Use when the user says "fill my daily note", "do my daily interview", "daily standup", or "wrap up my day".
---

You are interviewing the user to fill in their Obsidian daily note for a given day. Conduct a short, conversational interview — one topic at a time — then write the answers into the daily note, preserving anything already present. Do NOT invent activity; only record what the user tells you (plus context you surface for them to confirm).

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Grep, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Step 1 — Determine the target day
Get today's date with a shell `date +%Y-%m-%d` command first, then resolve the target day from the argument:
- **No argument or `today` → today.**
- **A specific `YYYY-MM-DD` → that date.** If the argument isn't a valid date, stop and ask for a valid one.

State the resolved target day at the start of the run.

## Step 2 — Read what's already there
1. Read the daily note at `6. Daily/<target-date>.md`. If it doesn't exist, use the template at `0. Templates/6. Daily notes template.md` as the starting structure and fill in the frontmatter (`date:`, and `weekly-plan:` copied from the most recent prior daily note if one exists).
2. Note which sections already have real content vs. template placeholders (e.g. "Main obj 1", "Loop 1", "N/A", empty `- `). You will augment, never blow away, real content.
3. Skim the two most recent prior daily notes (`6. Daily/`, newest first) so you know the current Main objectives / projects and which Open loops were still open — carry-over loops that weren't checked off should be raised in the interview.

## Step 3 — Pull the day's Linear activity
Gather everything the user touched in Linear on the target day, present it grouped for confirmation, then carry the **confirmed** items into the write in Step 6. Never write a Linear item into the note without the user confirming it — present first, write later.

If Linear is unavailable or every query returns nothing, say so briefly and skip to Step 4.

**Resolve the user and the day window first.** Get the user's id via `mcp__claude_ai_Linear__get_user` with query `"me"`. Get the local UTC offset with `date +%z` so you can map Linear's UTC (`...Z`) timestamps to the user's local calendar day: an activity belongs to the target day when its timestamp, converted to local time, falls on `<target-date>`.

**Run these queries (all via `mcp__claude_ai_Linear__list_issues`, `limit: 250`):**
1. **Assigned + touched** — `assignee: "me"`, `orderBy: "updatedAt"`, `updatedAt: "<target-date>"` (day start, as a cushion query "after"). Keep issues whose local `updatedAt` date is the target day.
2. **Created by me** — `createdAt: "<target-date>"`, `orderBy: "createdAt"` (no assignee filter). Keep issues where `createdById` == the user's id **and** local `createdAt` date is the target day. Merge into set 1, deduping by issue id.

**Classify each kept issue** (an issue can land in more than one bucket) using its timestamps, counting only those whose local date is the target day:
- **Done** — `completedAt` on the target day.
- **Started / In Review** — `startedAt` on the target day, or `statusType` is `started` and it entered that state today.
- **Canceled** — `canceledAt` on the target day.
- **Created** — from query 2.
- **Updated (no state change)** — `updatedAt` on the target day but none of the above (priority/description/label/assignee edits, etc.).

**Comments I made** — for each issue in the merged candidate set, call `mcp__claude_ai_Linear__list_comments` with that `issueId`, `orderBy: "updatedAt"`, and keep comments whose author is the user (match the user's id) with a local `createdAt`/`updatedAt` date on the target day. Note this limitation aloud if relevant: this only finds comments on issues the user is assigned to or created — Linear has no comments-by-author query, so a comment on some other issue won't surface here.

**Present** the results as a compact grouped list — e.g. `Done`, `Started / In Review`, `Created`, `Updated`, `Commented` — each line `IDENTIFIER — title` with the URL, and ask the user to confirm which to log and correct anything mislabeled. Hold the confirmed set for Step 6; use it in Step 5 so you don't re-ask about work Linear already surfaced.

## Step 4 — Check Granola for un-captured meetings
Find meetings that happened on the target day but don't yet have a note in `5. Meetings/`:
1. List the target day's Granola meetings — use `mcp__claude_ai_Granola__list_meetings` (or `mcp__claude_ai_Granola__query_granola_meetings`) and keep only those whose date is the target day. If Granola is unavailable or returns nothing, say so briefly and skip to Step 5.
2. Glob `5. Meetings/` for existing notes dated the target day (filenames follow `<YYYY-MM-DD> - <Title>.md`). A Granola meeting is **un-captured** if no existing note has the same date and a matching title (compare case-insensitively, ignoring punctuation; treat a clear title match as captured even if not identical).
3. Present the un-captured meetings as a short list and ask whether to create notes for them. For each one the user wants, create the note by following the **`meeting-from-granola`** skill's conventions — you already have the Granola meeting ID from step 1, so call `mcp__claude_ai_Granola__get_meetings` with that ID directly (no URL needed), write `5. Meetings/<YYYY-MM-DD> - <Title>.md`, and update the Meetings MOC. Do not create notes the user didn't confirm.
4. Any meeting captured today (existing or newly created) is context for the interview — surface its decisions/commitments when asking about tasks and open loops.
5. **Record decisions taken in those meetings into `3. Decisions/`.** For each meeting captured for the target day (an existing note or one you just created), read its `## Decisions` section. Treat as a candidate only a **substantive** decision — a choice with lasting effect (vendor/tooling/architecture/process/scope), not a routine task, scheduling note, or restatement of a commitment. For each candidate, glob `3. Decisions/` and skip any that already has a note on the same subject (don't duplicate). Present the remaining candidates and ask the user which warrant a Decision note; create only the ones they confirm. For each confirmed decision, write `3. Decisions/<YYYY-MM-DD> - <Short title>.md` (date = the meeting date) using the vault's Decision template at `0. Templates/3. Decision template.md`:
   - Set `status: decided` for a decision that was actually made; `status: pending` if it was tabled/still open. Add lowercase topic tags after `decisions`.
   - Fill **Decision** (what was decided), **Why** (rationale if stated), **Alternatives considered**, **Decision Owner** (`[[Name]]`), **Revisit Trigger**, and **AI Recall**. Omit a section only when there's genuinely nothing for it, matching how existing decision notes are written — don't fabricate.
   - Reference the source meeting inline with a `- Decided in [[<YYYY-MM-DD> - <Meeting Title>]]` line so the decision traces back to its meeting.
   - Add `- [[<YYYY-MM-DD> - <Short title>]]` to `3. Decisions/Decisions MOC.md`.

## Step 5 — Conduct the interview
Ask about the three topics below **one at a time**, in order. Wait for the user's answer to each before moving on. Keep questions short and specific; ask a brief follow-up only when an answer is ambiguous (e.g. which project a ticket belongs to, or whether a blocker is now resolved). Don't over-interrogate — if they say "that's it" for a topic, move on.

Lead with the Linear activity the user confirmed in Step 3 — treat those as already-captured tickets/tasks and ask only what Linear can't tell you (which project a ticket rolls up to if unclear, whether in-progress work is blocked, what's still open). Don't re-ask the user to list work you already surfaced.

1. **Tickets & tasks done** — "What tickets or tasks did you work on or finish today?" For each, capture: the work, whether it's done or in progress, the project it rolls up to, and any ticket/PR link. Map these to the project-grouped Main objectives / Side objectives structure.
2. **Blocking issues** — "Any blockers or issues that came up today — things that stopped or slowed you?" For each, capture what it is, who owns unblocking it (`[[Name]]`), and whether it's now resolved or still blocking.
3. **Open loops remaining** — "What's still open — loops you need to pick back up?" Capture each as a to-do, attributing an owner with `[[Name]]` where relevant. Also confirm the status of any carry-over open loops you saw in Step 2 (resolved → check off; still open → keep).

## Step 6 — Write the note
Write/Edit `6. Daily/<target-date>.md` following the Daily template structure exactly. Preserve all existing real content; merge new answers in.

Template structure:
```
---
type: daily
weekly-plan: <"[[... - Personal weekly planning]]" if known>
date: <target-date>
tags:
  - daily
---
# Daily summary

## Main objectives

- [[<YYYY-MM-DD> - Project name]]
	- [x] Completed task / note about progress
	- [ ] Task still in progress

## Side objective 1

- <side work>

## Side objective 2

- N/A

## Non-negociable operational tasks

- [ ] Update Obsidian

---
## Decisions / Signals

- <decisions made or notable signals; also resolved blockers worth remembering>

## Open loops

- [ ] <open loop>. [[Owner]] is handling this
- [x] <loop resolved today>
```

Conventions (match the existing daily notes exactly):
- **Tickets/tasks done** go under Main objectives, grouped as sub-bullets beneath the `[[YYYY-MM-DD - Project]]` note they belong to. Use `- [x]` for done, `- [ ]` for in progress, or a plain `- ` sub-bullet for a progress note. Work that doesn't fit a tracked project goes under a Side objective. Include PR/ticket URLs inline when given.
- **Projects and people are `[[Wikilinks]]`.** Reference projects by their existing note name in `2. Projects/` and people by first name as they appear in `1. People/` (check those folders; if the entity has no note yet, still write the `[[wikilink]]` — vault-groom will create the orphan later).
- **Blocking issues:** if still blocking, write them as `- [ ]` items under Open loops with the owner (`[[Name]]`); if found and resolved today, record them under Decisions / Signals (and/or as a checked `- [x]` loop). Make the blocker itself explicit.
- **Open loops** are `- [ ]` (open) / `- [x]` (resolved), attributing `[[Owner]]` where relevant, matching the "…[[Name]] is handling this" phrasing already in use.
- Keep "Non-negociable operational tasks" as-is; only check off "Update Obsidian" if the user confirms it's done.
- Leave a section with genuinely nothing as `N/A` (side objectives) or a single `- ` placeholder, matching the template — don't fabricate filler.

## Step 7 — Confirm and report
Show the user the filled sections (or a concise summary) and confirm it's written. Then report:
- **Daily note:** `6. Daily/<target-date>.md` (created or updated)
- **Linear:** counts of activity surfaced and confirmed — N done, N started/in review, N created, N updated, N commented (or "none")
- **Meetings:** un-captured Granola meetings found, and which meeting notes you created (or "none")
- **Decisions:** which Decision notes you created in `3. Decisions/` from the day's meetings (or "none")
- **Logged:** counts — N tickets/tasks, N blockers, N open loops
- **New `[[wikilinks]]`:** flag any people/projects referenced that don't yet have a note (suggest running `vault-groom` to create them)
Do NOT commit or push — that's `vault-groom`'s job. If the user asks, offer to run it.
