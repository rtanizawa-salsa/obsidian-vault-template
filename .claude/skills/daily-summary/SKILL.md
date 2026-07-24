---
name: daily-summary
argument-hint: "[today (default) | YYYY-MM-DD]"
description: Interview the user to fill in the current day's Obsidian daily note. Asks, one topic at a time, about tickets/tasks done, blocking issues found during the day, and open loops still remaining, then writes the answers into `6. Daily/<date>.md` using the vault's Daily template — preserving anything already there. Use when the user says "fill my daily note", "do my daily interview", "daily standup", or "wrap up my day".
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

## Step 3 — (Optional) Jog memory from Linear
To help the user recall what they touched, you MAY pull their recent Linear activity for the target day before interviewing — use `mcp__claude_ai_Linear__list_issues` filtered to issues assigned to / updated by them around the target date (resolve their user via `mcp__claude_ai_Linear__list_users` if needed). Present these as a short "Here's what I see you touched — anything to log?" list. Keep it lightweight; if Linear is unavailable or returns nothing, skip silently and just interview. Never write Linear items into the note without the user confirming them.

## Step 4 — Conduct the interview
Ask about the three topics below **one at a time**, in order. Wait for the user's answer to each before moving on. Keep questions short and specific; ask a brief follow-up only when an answer is ambiguous (e.g. which project a ticket belongs to, or whether a blocker is now resolved). Don't over-interrogate — if they say "that's it" for a topic, move on.

1. **Tickets & tasks done** — "What tickets or tasks did you work on or finish today?" For each, capture: the work, whether it's done or in progress, the project it rolls up to, and any ticket/PR link. Map these to the project-grouped Main objectives / Side objectives structure.
2. **Blocking issues** — "Any blockers or issues that came up today — things that stopped or slowed you?" For each, capture what it is, who owns unblocking it (`[[Name]]`), and whether it's now resolved or still blocking.
3. **Open loops remaining** — "What's still open — loops you need to pick back up?" Capture each as a to-do, attributing an owner with `[[Name]]` where relevant. Also confirm the status of any carry-over open loops you saw in Step 2 (resolved → check off; still open → keep).

## Step 5 — Write the note
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

## Step 6 — Confirm and report
Show the user the filled sections (or a concise summary) and confirm it's written. Then report:
- **Daily note:** `6. Daily/<target-date>.md` (created or updated)
- **Logged:** counts — N tickets/tasks, N blockers, N open loops
- **New `[[wikilinks]]`:** flag any people/projects referenced that don't yet have a note (suggest running `vault-groom` to create them)
Do NOT commit or push — that's `vault-groom`'s job. If the user asks, offer to run it.
