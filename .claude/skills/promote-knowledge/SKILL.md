---
name: promote-knowledge
argument-hint: "[all (default) | YYYY-MM-DD | topic or keyword]"
description: Promote knowledge candidates out of the inbox into durable Knowledge notes. Reads `7. Knowledge/Knowledge candidates.md`, groups candidates by topic, and either enriches the existing `7. Knowledge/` note on that topic or creates a new one at the right altitude — a subject page that accumulates facts over time, never a page per implementation detail. Updates the Knowledge MOC and clears promoted lines from the inbox. Use when the user says "promote knowledge candidates", "promote that candidate", "clear the knowledge inbox", "turn the candidates into knowledge notes", or names a topic to promote.
---

You are promoting distilled facts out of the knowledge inbox into the user's durable Knowledge layer. `vault-groom` deliberately refuses to touch Knowledge notes because they are hand-curated — this skill is the deliberate act it defers to. Run fully automatically — read, group, write, and merge without asking for approval — then report what you did in chat.

## Vault location
The vault is the current working directory — the root of the user's Obsidian vault. Use the file tools (Read, Write, Edit, Grep, Glob) relative to it. If those tools can't access it, the vault folder connection was lost — say so and stop, and ask the user to reconnect their Obsidian vault folder.

## Step 1 — Read the inbox and the existing Knowledge layer
1. Locate the candidates inbox by Glob (`7. Knowledge/*Knowledge candidates*.md`) rather than a hard-coded path — the filename may carry a sort prefix. If there is no inbox file or it holds no unchecked candidates, say so and stop.
2. Glob `7. Knowledge/` and **Read every existing Knowledge note in full.** You cannot judge "does a page for this already exist?" or "what altitude do this vault's pages sit at?" from filenames alone.

## Step 2 — Select the candidates to promote
Resolve from the argument passed to the skill:

- **No argument, or `all` → every unchecked (`- [ ]`) candidate** in the inbox.
- **A date (`YYYY-MM-DD`) → the candidates under that date heading.**
- **Anything else → a topic/keyword filter.** Select candidates whose fact, proposed target, or source matches it.

If the filter matches nothing, say so and list the available candidates instead of guessing.

## Step 3 — Group candidates into topics
Cluster the selected candidates by **subject**, not by source note or by the `**New note:**` title vault-groom suggested. Several candidates from different days and different meetings routinely belong to one page — that accumulation *is* the page. Each cluster is promoted as a single unit of work.

## Step 4 — Decide the target page and its altitude
This is the judgment call the skill exists for. **A Knowledge page is a subject someone would look up** — `Release process`, `API rate limits`, `Onboarding checklist` — **not a record of one implementation decision.** Individual decisions are already captured in `3. Decisions/`; what belongs here is the durable understanding they accumulate into.

For each cluster, pick the target in this order of preference:

1. **An existing Knowledge note covers the subject → enrich it.** Always prefer this. "Covers the subject" is about the topic, not the exact fact — a new edge case for `Release process` belongs *inside* `Release process`.
2. **No existing note → create the page for the broader subject** the candidates are instances of, seeded with everything you can gather on that subject (Step 5).
3. **Neither fits → park the candidate** (Step 7).

Altitude test for any title you are about to create: *could this page plausibly accumulate five or more distinct facts over the next year?* If not, it is too narrow — go one level up and make the candidate a section, row, or edge case inside the broader page.

Signals a proposed title is too narrow: it names a single decision, ticket, class, function, or field; it reads as "X does Y in Z case"; it only makes sense to someone who saw the originating meeting. The `**New note:**` line in a candidate is a *hint from grooming*, not an instruction — override it whenever it sits too low, and say so in your report.

Never create one page per candidate.

## Step 5 — Gather depth before creating a new page
A new page must stand on its own, not read as a stub. Before writing one, sweep the vault for everything on that subject and fold it in:
- other candidates in the inbox on the same subject (promote them in the same pass and clear their lines too);
- the `Source:` notes the candidates link to, read in full — plus their sibling `3. Decisions/`, `5. Meetings/`, `2. Projects/`, and `4. Companies/` notes that Grep surfaces for the subject's key terms;
- context already sitting in other Knowledge notes that belongs on this page instead.

Everything you write must be traceable to vault content. **Never fabricate, infer beyond what the notes support, or pad with outside knowledge** to make a page look fuller. If the gathered material yields only a bare summary and one thin detail, the subject is not ready — park it (Step 7) rather than committing a stub.

## Step 6 — Write the page
**Enriching an existing note** — you are adding to a curated document, not rewriting it:
- Preserve all existing hand-written content, structure, and voice. Integrate each fact where it belongs (extend a table, add a bullet, add an edge case) and only add a new `## <Aspect>` section when nothing fits.
- If a candidate contradicts what the note already says, keep both and flag it inline — `> ⚠️ 2026-07-27 [[source]] says X; note said Y — reconcile` — never silently overwrite.
- Add the candidate's `Source:` `[[wikilink]]` to the note's source list.

**Creating a new note** — `7. Knowledge/<Subject>.md`, following `0. Templates/7. Knowledge template.md`:
```
---
type: knowledge
area: <e.g. Engineering, Process, Product — omit if genuinely unclear>
tags:
  - knowledge
  - <lowercase topic tags>
---
# <Subject>

## Summary

<2–4 lines: what this subject is and why it matters. Written for someone with no memory of the meetings.>

## Details

<The accumulated facts, organized for lookup rather than narrative — sub-headings per aspect,
tables for canonical values, blockquotes for verbatim rules. Leave the structure open enough
that future promotions slot in without a rewrite.>

## Source
- <[[wikilinks]] to the vault notes each fact came from>
- Type: <meeting | decision | source distillation>
- Origin: <person, document, or system the knowledge came from>
- Processed: <YYYY-MM-DD — get today's date with a shell `date` command>
```
Sources here are permanent vault notes, so `[[wikilinks]]` are correct — unlike `distill-source`, which cites transient files as plain text. Cross-link into the rest of the vault where natural: `[[Company]]`, `[[Person]]`, related `[[Knowledge note]]`.

## Step 7 — Update the inbox
For every candidate you handled, edit the inbox file:
- **Promoted** → delete the line and its sub-bullets. Delete the date heading too if it is now empty.
- **Already covered** by an existing note (the fact was a duplicate) → delete the line and report it as already covered; do not rewrite the page.
- **Parked** (too narrow, and no broader page is ready yet) → leave the line in place and append a sub-bullet: `- ↳ Parked <YYYY-MM-DD>: waiting for more facts on <subject>` so the next run knows not to re-litigate it.

Never delete a candidate you did not actually promote, cover, or park.

## Step 8 — Update the Knowledge MOC
Add every newly created note to `7. Knowledge/0. Knowledge MOC.md` as a `[[wikilink]]` list item. Notes you only enriched are already listed — don't re-add them.

## Step 9 — Log the run
Append a one-line summary to `log.md` under today's `## YYYY-MM-DD` heading (newest first) — e.g. `promoted <n> knowledge candidates → [[note]] created, [[note]] enriched`. Create the date heading if it doesn't exist. Do **not** commit or push — leave the vault's git state for the user or the next `vault-groom` run to sync, unless the user explicitly asks you to commit.

## Output (report in chat)
Report concisely:
- **Candidates selected:** how many, and the filter used
- **Notes created:** each new `7. Knowledge/` note, the candidates it absorbed, and a one-line summary of the subject
- **Notes enriched:** each existing note, and exactly what was added to it
- **Altitude calls:** any candidate whose suggested `**New note:**` title you overrode, and the broader subject you used instead
- **Already covered:** candidates dropped because an existing note already held the fact
- **Parked:** candidates left in the inbox, with why
- **Conflicts flagged:** any place a candidate contradicted an existing note (or "none")
- **MOC updated:** yes/no
- **Inbox remaining:** how many unchecked candidates are still in the inbox
If a section had no activity, say "none" for it. Keep it scannable.
