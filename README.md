# Obsidian PKM Vault — Template

A template for a personal knowledge base (Obsidian vault) optimized for both human and LLM use. Notes are small, single-subject, typed, and cross-linked so an assistant can read a few system files and then operate the vault correctly.

This repository is a **structural template populated with fictional example data**. Every person, company, project, and meeting below is invented — swap them for your own. The canonical description of structure and conventions lives *inside* the vault (see `schema.md`); this README is the outside-the-vault overview for anyone browsing the repo.

## Setup

No command line required — but a terminal path is provided too. Pick the option that fits you.

### 1. Create your own **private** copy

This becomes *your* vault — you'll add your real notes to it, so it **must be private**.

> ⚠️ **Don't use the Fork button.** A fork of a public repo is always **public**, and GitHub removes the option to change a fork's visibility — so you *cannot* make a fork private afterward, and your notes would be exposed. Use one of the options below instead; both let you choose Private.

**Option A — GitHub website (no command line)**

1. On this repository's GitHub page, click the green **Use this template** button (top right) → **Create a new repository**.
2. Choose the **Owner** (you or your team) and a **Repository name** (e.g. `my-vault`).
3. Set **Visibility** to **🔒 Private** — this is the important one; it keeps your notes private.
4. Click **Create repository**.

**Option B — Command line ([GitHub CLI](https://cli.github.com))**

```bash
gh repo create my-vault --private --clone --template rtanizawa-salsa/obsidian-vault-template
```

This creates the private repo *and* clones it to your computer in one step.

Either way you end up with your own private repo and your own commit history — from here you commit your own notes.

### 2. Get it onto your computer

- **If you used Option A (website):** clone it with **[GitHub Desktop](https://desktop.github.com)** — a graphical app, no terminal needed. Install it, sign in, then **File → Clone repository** and pick your new `my-vault` repo. It also handles committing and syncing with buttons instead of git commands. *(Prefer the terminal? `git clone <your-private-repo-url>`.)*
- **If you used Option B (CLI):** already done — `--clone` put it on your computer.

### 3. Open it in Obsidian

Install **[Obsidian](https://obsidian.md)**, then **Open folder as vault** and select the folder you just cloned.

### 4. Enable the templates

In Obsidian: **Settings → Community plugins → Turn on community plugins**, then install **Templater** (required — it drives `0. Templates/`). Then, in **Settings → Templater**, set the template folder to `0. Templates/`. Optional plugins are listed under [Obsidian community plugins](#obsidian-community-plugins) below.

### 5. Make it yours

Delete the fictional example notes (people, companies, projects, meetings, dailies) and start writing your own, keeping the frontmatter and section conventions from [schema.md](schema.md). Keep the system files (`index.md`, `schema.md`, `log.md`) and the per-folder MOCs.

### 6. (Optional) Wire up automation

Pair the vault with Claude Code for maintenance — see [Automating with Claude Code](#automating-with-claude-code-skills).

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

## Obsidian community plugins

This template pairs with the community plugins below. Plugin config is not shipped in this repo — install them from Obsidian's community plugin browser. Only **Templater** is required (it drives `0. Templates/`); the rest are optional quality-of-life picks.

| Plugin | Required | Description |
| --- | --- | --- |
| [Templater](https://github.com/SilentVoid13/Templater) | Yes | Advanced templating and automation using handlebars-like syntax. Drives the note templates in `0. Templates/`. |
| [Flexplorer](https://github.com/obsidian-flexplorer/obsidian-flexplorer) | No | Enhances the file explorer with custom sorting, pinning, and hiding. |
| [Terminal](https://github.com/polyipseity/obsidian-terminal) | No | Integrates consoles, shells, and terminals. |
| [Show Hidden Files](https://github.com/polyipseity/obsidian-show-hidden-files) | No | Reveals hidden dotfiles and all file types in the file explorer. |

## Automating with Claude Code (skills)

This vault pairs well with [Claude Code](https://claude.com/claude-code) for maintenance. It ships with two example skills under `.claude/skills/` — invoke them by slash command in a Claude Code session pointed at this folder, or adapt them / add your own:

| Skill | Invoke | What it does |
| --- | --- | --- |
| **vault-groom** | `/vault-groom [today \| yesterday \| YYYY-MM-DD]` | Grooms a day's notes: creates orphan notes for newly referenced people/projects/companies, consolidates duplicates, updates the MOCs, and flags strategic items. |
| **meeting-from-granola** | `/meeting-from-granola <url>` | Turns a [Granola](https://granola.ai) meeting URL into a note in `5. Meetings/` using the Meeting template, then updates the Meetings MOC. *Requires the Granola connector* — swap in your own meeting source if you don't use it. |

### Scheduling a skill (Claude Code desktop)

Run a skill automatically on a recurring schedule as a **local scheduled task** (a "routine" that runs on your machine while the desktop app is open):

1. In Claude Code desktop, open **Routines** in the sidebar → **New routine** → **Local**.
2. Fill in:
   - **Name**: `vault-groom-daily`
   - **Instructions**: `/vault-groom yesterday`
   - **Schedule**: **Weekdays**, 9:00 AM
   - **Folder**: this vault folder (it must be trusted first)
   - **Permission mode**: Ask (switch to Allow after a successful test run)
3. Click **Create**, then **Run now** once and approve the tools it uses — future runs auto-approve the same tools.

Local tasks run only while the desktop app is open and the computer is awake; the app checks schedules every minute. You can also just ask Claude in any session — *"schedule `/vault-groom` to run every weekday at 9am"* — and it will set it up. For runs that fire even when your machine is off, use **cloud routines** via the `/schedule` command instead.
