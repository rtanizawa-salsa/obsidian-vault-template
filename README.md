# Obsidian PKM Vault — Template

A template for a personal knowledge base (Obsidian vault) optimized for both human and LLM use. Notes are small, single-subject, typed, and cross-linked so an assistant can read a few system files and then operate the vault correctly.

This repository is a **structural template populated with fictional example data**. Every person, company, project, and meeting below is invented — swap them for your own. The canonical description of structure and conventions lives *inside* the vault (see `schema.md`); this README is the outside-the-vault overview for anyone browsing the repo.

## Entry points

Start with these three system notes at the vault root:

- **[index.md](index.md)** — the human/LLM entry point and current map of the vault (Maps of Content, people, projects, recent meetings).
- **[schema.md](schema.md)** — how the vault is structured: folder taxonomy, note types, required frontmatter, and conventions. Read this before creating or editing notes.
- **[log.md](log.md)** — append-only record of meaningful changes, newest first.

## Folder taxonomy

| Folder | Note `type` | Holds |
| --- | --- | --- |
| `0. Templates/` | (templates) | One Templater source note per note type |
| `1. People/` | `person` | One note per individual (`<FirstName>.md`) |
| `2. Projects/` | `project` | One note per project (`YYYY-MM-DD - <Title>.md`) |
| `3. Decisions/` | `decision` | One decision record per note |
| `4. Companies/` | `companies` | One note per external org/vendor |
| `5. Meetings/` | `meeting` | One note per meeting (`YYYY-MM-DD - <Title>.md`) |
| `6. Daily/` | `daily` | One note per day (`YYYY-MM-DD.md`) |
| `7. Knowledge/` | reference | Durable reference notes, policies, facts |

## Conventions

- **Linking:** reference other notes with `[[Wikilink]]` syntax; prefer linking over duplicating facts. People are linked by first name, matching the filename.
- **MOCs:** each content folder has a `<Name> MOC.md` acting as its index/hub.
- **Dates:** ISO `YYYY-MM-DD`; dated note types are prefixed by their anchoring date.
- **Frontmatter:** every note starts with YAML frontmatter including `type` and lowercase `tags`. See [schema.md](schema.md) for the required fields per type.
- **One subject per note.** Split rather than grow a note past its single subject.

## Using this template

1. Clone or use the repo as a template.
2. Open the folder as a vault in [Obsidian](https://obsidian.md).
3. Delete the fictional example notes and start writing your own, keeping the frontmatter and section conventions from `schema.md`.
4. The templates in `0. Templates/` are wired for the [Templater](https://github.com/SilentVoid13/Templater) community plugin.
