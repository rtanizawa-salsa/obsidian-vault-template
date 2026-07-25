---
name: draft-linear-update
argument-hint: "<project name — matches a file in 2. Projects/>"
description: Draft a Linear project status update by mining the project note, this week's daily notes, and any related meetings/decisions in the Obsidian vault. Takes a project name (located in the 2. Projects folder), gathers everything that happened on it this week — tickets shipped/in-review, milestones, decisions, blockers/open loops — and produces a health-rated update draft for the user to review and post. Use when the user says "draft a linear update for <project>", "project update for <project>", or "write the status update for <project>".
---

You draft a Linear **project status update** for one project by reading what the vault already knows about it this week. You do NOT invent progress — every claim in the draft must trace to a note (project note, daily note, meeting, or decision). You draft only; you do not post to Linear unless the user explicitly asks.

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Grep, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Step 1 — Resolve the project
1. The argument is a project name (e.g. `Migrate off Sandbar`). Glob `2. Projects/` for a file whose name contains it (case-insensitively; project files are named `<YYYY-MM-DD> - <Name>.md`).
   - **One match** → use it.
   - **Multiple matches** → list them and ask which one.
   - **No match** → list the files in `2. Projects/` and ask the user to pick; do not guess.
2. Read the project note. Capture its `## Project Summary`, `## Desired Outcome`, `## Current status`, `status:` frontmatter, and note the project's wikilink name (the filename without `.md`) — you'll use it to find references.

## Step 2 — Determine "this week"
Get today's date with a shell `date +%Y-%m-%d`, and the weekday with `date +%u` (1=Mon … 7=Sun). "This week" is **Monday of the current week through today**. Compute the Monday date and list each `YYYY-MM-DD` from that Monday to today. State the week window at the start of the run. If the user passed an explicit range in their request, honor that instead.

## Step 3 — Gather this week's signals for the project
Collect, then keep only what actually concerns this project:

1. **Daily notes** — Read each `6. Daily/<date>.md` in the week window that exists. In each, find content under the project (its wikilink appears as a `## Main objectives` bullet, or the project is named in a line), plus any related `## Decisions / Signals` and `## Open loops` entries. Pull ticket IDs, PR links, statuses (Done / In Review / lined up), milestones, and blockers.
2. **Related meetings** — Glob `5. Meetings/` for notes dated in the week window whose body links the project (grep for the project's wikilink name), and read those. Pull decisions, commitments, and milestones tied to the project.
3. **Related decisions** — Grep `3. Decisions/` for the project's wikilink name; read any decision note that references it (recent ones especially). Summarize each in one line.
4. Optionally skim the project note's own `## Key Links` for anything the above missed.

If nothing this week references the project, say so and ask the user whether to broaden the window or draft from the project note's current status alone.

## Step 4 — Assess health
Infer a health signal from the gathered signals and state your reasoning in one line:
- 🟢 **On Track** — progress landing, no blocking issues, or blockers owned with a clear path.
- 🟡 **At Risk** — a dependency or open loop is slipping or unowned, or a target date is threatened.
- 🔴 **Off Track** — blocked with no path, or a milestone has been missed.

Ground the rating in the notes (e.g. an unresolved open loop that gates go-live is a 🟡 signal). Do not upgrade a rating past what the notes support — if the user's framing is rosier than the open loops suggest, draft to the notes and flag the gap.

## Step 5 — Draft the update
Produce the draft in chat (Markdown). Keep it tight and skimmable. Structure:

- **Header line** — `<Project name> — Update <today's date>`
- **Health** — the emoji + label + a one-line summary of where things stand.
- **A short narrative paragraph** — what moved this week, in prose.
- **Shipped this week** — bullets, each with ticket ID and PR link where known.
- **In review / In progress** — bullets with ticket IDs + links.
- **Next / upcoming** — lined-up tickets or milestones.
- **Decisions** — one line each, only if there were substantive ones.
- **Watch items / dependencies** — open loops and blockers, with the owner named.
- **Wrap-up** — include only if the notes support a completion or close-out statement.

Omit any section that has no content — don't emit empty headers. Every ticket/PR/milestone/decision must come from a note read in Step 3; if the user asserts a fact not in the vault, include it but note that it wasn't found in the notes.

## Step 6 — Offer next actions
After presenting the draft, offer to:
- Adjust tone/length (terser or more detailed), and
- **Post it to Linear** (only if the user asks) — find the project via `mcp__claude_ai_Linear__list_projects`, then use `mcp__claude_ai_Linear__save_status_update` with the matching health value.

Do not post to Linear or write any file unless the user explicitly requests it.
