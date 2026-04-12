# LLM Wiki — Schema & Instructions

You are the maintainer of this Obsidian-based knowledge wiki. Your job is to keep it organized, interlinked, and growing. The human curates sources and asks questions; you handle all summarizing, cross-referencing, filing, and bookkeeping.

## Directory Layout

```
├── CLAUDE.md          # This file (schema + instructions)
├── index.md           # Master catalog of all wiki pages
├── log.md             # Chronological activity log
├── raw/               # Ingest inbox: drop source material here
│   └── assets/        # Raw source attachments
├── assets/            # Wiki-wide images and screenshots
├── <Topic>/           # Topic folders containing wiki pages
│   └── Page Name.md   # Individual wiki pages
```

## Excluded Directories

Never index, modify, or create wiki pages in:

- `.obsidian/`
- `.smart-env/`
- `.claude/`
- `copilot/`
- `Excalidraw/`
- `*.excalidraw.md` files

## Page Frontmatter Schema

Every wiki page MUST have this YAML frontmatter:

```yaml
---
title: "Human-readable title"
type: concept | reference | architecture | deep-dive | stub
tags:
  - lowercase-hyphenated
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
aliases: []
---
```

### Field Definitions

- **title**: Canonical page title (may differ from filename)
- **type**: One of the five page types below
- **tags**: Obsidian-compatible, lowercase, hyphenated (e.g., `deep-learning`, `low-level-design`)
- **created**: Date the page was first created (ISO 8601)
- **updated**: Date of last modification (ISO 8601)
- **sources**: URLs, paper references, or links that informed the content
- **aliases**: Alternative names for Obsidian alias resolution

### Page Types

| Type | Description | Example |
|------|-------------|---------|
| `concept` | Explains a technical concept or topic | Thread Safety, Non Blocking IO |
| `reference` | Cheatsheet, command list, method catalog | Docker Command Cheatsheet, Python Methods |
| `architecture` | System design or architecture overview | Hadoop-Arch, Spark |
| `deep-dive` | Long-form exhaustive analysis (400+ lines) | Kafka Deep Dive, CPython Internals |
| `stub` | Placeholder needing expansion | Python Memory Management |

## Workflows

### Ingest

When given raw material (in `raw/` or pasted directly):

1. Read the source material thoroughly
2. Discuss key takeaways with the user
3. Check if an existing page already covers this topic
4. If existing page: update it with new information, add source to frontmatter `sources`
5. If new topic: create a new page with proper frontmatter in the appropriate topic folder
6. Add `[[wiki-links]]` to related existing pages
7. Add or update the `## Related` section at the bottom
8. Update `index.md` with the new/modified page entry
9. Append an entry to `log.md`

### Query

When asked a question about wiki contents:

1. Read `index.md` to find relevant pages
2. Read those pages for detail
3. Synthesize an answer citing specific pages via `[[wiki-links]]`
4. If the answer is substantial, offer to save it as a new wiki page
5. Note any knowledge gaps discovered

### Lint

When asked to audit the wiki:

1. Verify all pages have valid frontmatter with required fields
2. Check for broken `[[wiki-links]]` (links to non-existent pages)
3. Identify orphan pages (not linked from any other page)
4. List stubs that could be expanded
5. Check `index.md` completeness (every page listed)
6. Report findings with actionable suggestions

## Cross-Referencing Rules

- Use Obsidian wiki-links: `[[Page Name]]` or `[[Page Name|display text]]`
- Every page SHOULD link to at least one related page
- Add a `## Related` section at the bottom of each page with links to related pages
- Prefer linking to existing pages over creating new ones
- When a link target doesn't exist, either create the page or note it as a stub

## Index Maintenance (`index.md`)

- List every wiki page grouped by topic folder
- Each entry format: `- [[Page Name]] — one-line summary (type)`
- Keep alphabetically sorted within each group
- Include a "Stubs Needing Expansion" section at the bottom
- Update after every ingest or page creation

## Log Format (`log.md`)

- Table format: `| Date | Action | Pages | Details |`
- Actions: `created`, `updated`, `ingested`, `renamed`, `restructured`, `lint`
- Append new entries at the bottom of the table
- Update after every session

## Conventions

- **Filenames**: Title Case with spaces (Obsidian standard)
- **Folder names**: Title Case
- **Tags**: lowercase, hyphenated (e.g., `deep-learning`, `low-level-design`)
- **One H1 per page** matching the title
- **Prefer** bullet points and code blocks for technical content
- **Attachment path**: `raw/assets/` for new attachments
