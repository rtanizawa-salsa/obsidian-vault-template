---
name: meeting-from-fathom
argument-hint: "[fathom-url | call-id] (omit to pick from recent meetings)"
description: Create an Obsidian meeting note from a Fathom recording. Given a Fathom call URL (e.g. https://fathom.video/calls/<id>), a share link, or a call ID — or nothing, in which case you pick from recent meetings — fetch the recording's summary, attendees, decisions, and commitments, then write a note in the "5. Meetings" folder using the vault's Meeting template and update the Meetings MOC. Use when the user pastes a Fathom link and asks to turn it into a meeting note, or says "meeting note from Fathom".
---

You are creating an Obsidian meeting note from a Fathom recording. Resolve the recording, fetch its summary (and transcript if needed), write the note using the vault's Meeting template, and update the Meetings MOC.

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Input
The user provides one of:
- A Fathom call URL — `https://fathom.video/calls/<id>` — or a share link (`/share/<token>`, including `/share/h/`, `/share/i/`, `/share/p/`, `/share/u/` variants).
- A bare numeric call ID (e.g. `737992396`).
- Nothing — in which case you list recent meetings and ask which one.

## Step 1 — Resolve the recording
Turn the input into a `recording_id` plus `title`, `date`, and `url`:
- **Pasted URL (call or share link)** → `mcp__claude_ai_Fathom__get_recording_by_url` with the URL.
- **Bare numeric call ID** → `mcp__claude_ai_Fathom__get_recording_by_call_id` with the number. If it returns "not found or access denied", retry the same number directly as a `recording_id` in Step 2's calls before concluding it's invalid.
- **No input** → `mcp__claude_ai_Fathom__list_meetings` (`max_pages: 3`), present the results as a short numbered list (`title | date | attendees`), and ask the user which to turn into a note. Do not proceed until they pick one.

If resolution returns nothing / access denied for a pasted URL or ID, stop and tell the user the recording couldn't be found (the link/ID may be wrong or not accessible to this account).

Capture from the resolution (and from `list_meetings` when that's the entry path): `recording_id`, `title`, `date`, `url`, `recorded_by`, and `calendar_invitees` — the invitees plus the recorder are your attendee list.

## Step 2 — Fetch the recording content
1. Call `mcp__claude_ai_Fathom__get_meeting_summary` with the `recording_id`. Use the AI summary and any action items it returns — action items map to **Commitments**.
2. If you need exact quotes, decisions aren't clear from the summary, or the summary is thin, also call `mcp__claude_ai_Fathom__get_meeting_transcript` with the `recording_id` and `url` and summarize from it. The transcript segments carry timestamped deep links (`[MM:SS](url?timestamp=secs)`) — you may cite a decision or commitment with one.

## Step 3 — Meeting note template
Read `0. Templates/5. Meeting template.md` and use it verbatim as the note's structure — same frontmatter keys, same sections, same order. Filename: `5. Meetings/<YYYY-MM-DD> - <Title>.md`, where the date is the meeting date and the title is the Fathom recording title (strip characters illegal in filenames: `/ \ : * ? " < > |`).

Fill it in as follows:
- `date` — the meeting date (`YYYY-MM-DD`).
- `tags` — keep `meetings`, then add relevant lowercase topic tags (e.g. `garnishments`, `payments`).
- `attendees` — a list of `"[[Name]]"` entries by first name (match how existing People notes are named — check `1. People/`). Derive them from `calendar_invitees` + `recorded_by`; use the person's name as it appears in the vault.
- **Summary** — 1–3 paragraphs from Fathom.
- **Decisions** — decisions made, one per line.
- **Commitments** — action items / follow-ups, ideally attributed with `[[Name]]`.
- **Preferences captured** — stated preferences or working styles.
- **Key Links** — always include `[Fathom recording](<meeting url>)`. You may add timestamped deep links for specific decisions/commitments.

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
- **Note:** flag any attendee names that don't yet have a People note, or anything you inferred vs. pulled directly from Fathom.
