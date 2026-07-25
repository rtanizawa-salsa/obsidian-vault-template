---
type: system
tags:
  - system
  - schema
---
# Vault Schema

Machine-readable map of how this vault is structured. Read this first before creating, editing, or querying notes so new content stays consistent with existing conventions.

## Purpose

A personal knowledge base optimized for both human and LLM use. Notes are small, single-subject, typed, and cross-linked. An LLM should be able to read [[schema]], [[index]], and [[log]] and then operate the vault correctly.

## Folder taxonomy

| Folder | Note `type` | Holds | Naming convention |
| --- | --- | --- | --- |
| `0. Templates/` | (templates) | Templater source notes, one per note type | `N. <Type> template.md` |
| `1. People/` | `person` | One note per individual | `<FirstName>.md` |
| `2. Projects/` | `project` | One note per project/initiative | `YYYY-MM-DD - <Title>.md` (start date) |
| `3. Decisions/` | `decision` | One note per decision record | `YYYY-MM-DD - <Title>.md` (decision date) |
| `4. Companies/` | `companies` | One note per external org/vendor | `<Company Name>.md` |
| `5. Meetings/` | `meeting` | One note per meeting | `YYYY-MM-DD - <Title>.md` |
| `6. Daily/` | `daily` | One note per day | `YYYY-MM-DD.md` |
| `7. Knowledge/` | (untyped/reference) | Durable reference notes, policies, facts | `<Topic>.md` |
| `8. MOC/` | (map) | Top-level Maps of Content | `<Name> MOC.md` |
| `9. Sources/` | (transient) | Raw source material staged for distillation into `7. Knowledge/`; deleted after processing | `<original filename>` |

## Note types and frontmatter

Every note starts with YAML frontmatter. Required fields per type:

- **person** — `type: person`, `status`, `role`, `team`, `tags: [people]`
  - Sections: `Linked Projects`, `Active Links`, `AI Recall Notes`
- **project** — `type: project`, `status`, `project leader`, `team members`, `tags`
  - Sections: `Project Summary`, `Desired Outcome`, `Key Links`, `Current status`, `Open Questions`, `Notes`
- **decision** — `type: decision`, `status: pending|accepted|superseded`, `tags: [decisions]`
  - Sections: `Decision`, `Why`, `Alternatives considered`, `Decision Owner`, `Revisit Trigger`, `AI Recall`
- **companies** — `type: companies`, `status`, `industry`, `tags: [companies]`
  - Sections: `Company Summary`, `Why this company matters`, `Relevant Links`, `Current relationship`, `Points of contact` (table), `AI Recall`
- **meeting** — `type: meeting`, `date`, `tags: [meetings]`, `attendees` (list of `[[wikilinks]]`)
  - Sections: `Summary`, `Decisions`, `Commitments`, `Preferences captured`, `Key Links`
- **daily** — `type: daily`, `weekly-plan`, `date`, `tags: [daily]`
  - Sections: `Daily summary` (Main objectives / Side objectives / Non-negotiable operational tasks), `Decisions / Signals`, `Open loops`

## Conventions

- **Linking:** reference other notes with `[[Note Name]]` wikilinks. Prefer linking over duplicating facts. People are linked by first name (`[[Alex]]`), matching the filename.
- **MOCs:** each content folder has a `... MOC.md` acting as its index/hub. `8. MOC/` holds cross-cutting maps that span folders.
- **Dates:** ISO `YYYY-MM-DD`. Dated note types are prefixed by their anchoring date.
- **`AI Recall` / `AI Recall Notes`:** free-text section where durable, LLM-relevant context is captured for future retrieval.
- **One subject per note.** Split rather than grow a note past its single subject.
- **Status vocabulary:** `active` / `pending` / `accepted` / `superseded` / `archived` as appropriate to the type.

## Entry points

- [[index]] — human/LLM entry point and current map of the vault
- [[log]] — append-only activity log of meaningful changes
- Per-folder MOCs listed in [[index]]
