# UK Library Availability Checker — Claude Code Skill

A skill for AI coding assistants that checks whether books are available at **UK public libraries**. Covers England, Scotland, Wales, and Northern Ireland.

## What it does

Given one or more book titles and a UK location (postcode or city), the skill:

1. Looks up book metadata via the [Open Library](https://openlibrary.org) API
2. Checks digital availability on the [Internet Archive](https://archive.org) (borrowable, browsable, or waitlist)
3. Provides a [WorldCat.org](https://search.worldcat.org) link — enter your postcode on the page to see which nearby libraries hold a physical copy
4. Links directly to your city's library catalogue (London, Manchester, Birmingham, Leeds, Bristol, Edinburgh, Glasgow, Liverpool, Cardiff, Northern Ireland)
5. Links to [Libby/OverDrive](https://libbyapp.com) for digital borrowing with your UK library card

## Usage

```
/check-library "The Opposite of Spoiled" near London
/check-library "Atomic Habits" near SW1A 2AA
/check-library "Dune" "1984" near Manchester
/check-library "How to Turn $100 into $1,000,000" near Edinburgh
```

**Argument format:** `"[book title(s)]" near [UK postcode or city]`

If you don't provide a location, the skill will ask for one.

> **UK only.** For other countries, visit [worldcat.org](https://www.worldcat.org) and enter your location.

---

## Installation

The skill lives in a single file: `SKILL.md`. Copy it into your AI tool's skills directory.

### Claude Code

```bash
mkdir -p ~/.claude/skills/check-library
cp SKILL.md ~/.claude/skills/check-library/SKILL.md
```

Then invoke it in any Claude Code session:

```
/check-library "Book Title" near [postcode or city]
```

### OpenClaw / other Claude-based CLIs

Skills are typically loaded from a `~/.claude/skills/<skill-name>/SKILL.md` path. If your tool follows the same convention:

```bash
mkdir -p ~/.claude/skills/check-library
cp SKILL.md ~/.claude/skills/check-library/SKILL.md
```

Check your tool's documentation for the exact skills directory location.

### OpenAI Codex CLI

Codex CLI uses a `~/.codex/skills/` directory. Copy the skill there:

```bash
mkdir -p ~/.codex/skills/check-library
cp SKILL.md ~/.codex/skills/check-library/SKILL.md
```

Then invoke it using your tool's slash-command prefix. The skill instructions are written in plain markdown and should be compatible with any tool that supports custom skills or system-prompt injection.

### Generic / Manual

If your tool doesn't have a skills directory, paste the contents of `SKILL.md` (everything after the `---` frontmatter) into a custom system prompt or instruction file.

---

## Supported cities

| City / Region | Catalogue |
|---|---|
| London (all boroughs) | LLC SirsiDynix consortium |
| Manchester | Spydus |
| Birmingham | Spydus |
| Leeds | Spydus |
| Bristol / South West | Libraries West |
| Edinburgh | Your Library Edinburgh |
| Glasgow | Encore |
| Liverpool | Spydus |
| Cardiff / Wales | Spydus |
| Northern Ireland | Libraries NI |
| Anywhere else in the UK | WorldCat.org (enter your postcode) |

---

## Data sources

| Source | Purpose |
|---|---|
| [Open Library Search API](https://openlibrary.org/developers/api) | Book metadata (title, author, ISBN) |
| [Internet Archive Loans API](https://archive.org/services/loans/) | Digital borrowing status |
| [WorldCat.org](https://search.worldcat.org) | Physical holdings at UK libraries |
| City-specific Spydus / Arena catalogues | Direct catalogue search links |
| [Libby/OverDrive](https://libbyapp.com) | eBook/audiobook lending via library card |

All sources are free and require no API key.

---

## Contributing

Pull requests welcome — especially for additional UK city catalogues or corrections to existing catalogue URLs.
