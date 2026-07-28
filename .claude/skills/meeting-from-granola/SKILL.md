---
name: meeting-from-granola
argument-hint: <granola-meeting-url>
description: Create an Obsidian meeting note from a Granola meeting URL. Given a Granola notes link (e.g. https://notes.granola.ai/t/<id>), fetch the meeting's summary, attendees, decisions, and commitments, then write a note in the "5. Meetings" folder using the vault's Meeting template and update the Meetings MOC. Use when the user pastes a Granola URL and asks to turn it into a meeting note, or says "meeting note from Granola".
---

You are creating an Obsidian meeting note from a Granola meeting URL. Fetch the meeting from Granola, write the note using the vault's Meeting template, and update the Meetings MOC.

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Input
The user provides a Granola meeting URL, typically of the form:
`https://notes.granola.ai/t/<uuid>-<suffix>` or `https://notes.granola.ai/d/<uuid>`

## Step 1 — Extract the meeting ID
The Granola meeting ID is the standard UUID embedded at the start of the last path segment: five hyphen-separated hex groups matching `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}`. Ignore any trailing `-<suffix>` after the UUID. If no valid UUID is present in the URL, stop and ask the user for a valid Granola meeting link.

## Step 2 — Fetch the meeting from Granola
Call `mcp__claude_ai_Granola__get_meetings` with the extracted UUID as the single `meeting_ids` entry. Use the returned AI-generated summary, private notes, attendees, title, and date.
- If you need exact quotes or the summary is thin, also call `mcp__claude_ai_Granola__get_meeting_transcript` with the same ID and summarize from it.
- If Granola returns nothing for the ID, stop and tell the user the meeting couldn't be found (the ID may be wrong or the meeting not synced).

## Step 3 — Meeting note template
Read `0. Templates/5. Meeting template.md` and use it verbatim as the note's structure — same frontmatter keys, same sections, same order. Filename: `5. Meetings/<YYYY-MM-DD> - <Title>.md`, where the date is the meeting date and the title is the meeting's Granola title (strip characters illegal in filenames: `/ \ : * ? " < > |`).

Fill it in as follows:
- `date` — the meeting date (`YYYY-MM-DD`).
- `tags` — keep `meetings`, then add relevant lowercase topic tags (e.g. `garnishments`, `payments`).
- `attendees` — a list of `"[[Name]]"` entries by first name (match how existing People notes are named — check `1. People/`). Use the person's name as it appears in the vault.
- **Summary** — 1–3 paragraphs from Granola.
- **Decisions** — decisions made, one per line.
- **Commitments** — action items / follow-ups, ideally attributed with `[[Name]]`.
- **Preferences captured** — stated preferences or working styles.
- **Key Links** — always include `[Granola notes](<original URL>)`.

Keep sections that have no content as the template's single `- ` placeholder.

## Step 4 — Write the note
Write the file to `5. Meetings/<YYYY-MM-DD> - <Title>.md`. If a note with that exact filename already exists, do not overwrite — report that it exists and ask the user whether to overwrite or create a variant.

## Step 5 — Update the Meetings MOC
Add a link to the new note in `5. Meetings/Meetings MOC.md` under the correct `## <YYYY-MM>` month heading (create the heading if the month is new). Keep entries sorted newest first within the month. Format: `- [[<YYYY-MM-DD> - <Title>]]`.

## Step 6 — Report
Report concisely in chat:
- **Note created:** the filename
- **Attendees:** the `[[wikilinks]]` used
- **MOC updated:** yes/no
- **Note:** flag any attendee names that don't yet have a People note, or anything you inferred vs. pulled directly from Granola.
