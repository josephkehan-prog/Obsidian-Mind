# Wiki Schema

This file defines the conventions, structure, and rules for this vault. Every agent operating on the vault must read this file at the start of every session before performing any operation.

---

## Directory Layout

```
vault/
├── inbox/          binary originals (.pdf .docx .pptx .epub .png …) — drop here for conversion
├── raw/            converted, cleaned markdown sources — immutable after creation
├── wiki/
│   ├── entities/   people, tools, products, organisations, projects
│   ├── concepts/   ideas, frameworks, methodologies, techniques
│   ├── comparisons/ side-by-side analyses of 2+ entities or approaches
│   └── queries/    filed Q&A answers derived from wiki knowledge
├── output/         generated reports, syntheses, exports
├── index.md        master catalog — one row per wiki page
└── log.md          append-only operation log
```

---

## Page Types

| Type | Directory | When to use |
|------|-----------|-------------|
| `entity` | `wiki/entities/` | A specific named thing: person, tool, project, company |
| `concept` | `wiki/concepts/` | An idea, methodology, pattern, or framework |
| `comparison` | `wiki/comparisons/` | Side-by-side analysis of 2+ entities/approaches |
| `query` | `wiki/queries/` | A substantive Q&A answer worth preserving |

**Threshold for creating a new page:**
- Content appears in 2+ sources, **or**
- Content is central and detailed in 1 source (≥3 distinct facts about it)

Do not create stub pages. A page must have enough content to stand alone.

---

## Required Frontmatter

Every wiki page must begin with this YAML block:

```yaml
---
title: <human-readable title>
type: entity | concept | comparison | query
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
sources: [raw/slug.md]
contradictions: false
---
```

`contradictions: true` — set when the page contains conflicting claims from different sources. Note both positions with dates and source citations inline.

---

## Tag Taxonomy

Use only tags from this list. Add new tags to this file when a genuine new category emerges.

| Tag | Use for |
|-----|---------|
| `ai` | Artificial intelligence systems, models, research |
| `tooling` | Developer tools, IDEs, CLIs, build systems |
| `workflow` | Processes, pipelines, procedures |
| `architecture` | System design, structural patterns |
| `research` | Academic papers, studies, findings |
| `person` | Individual people (combine with domain tag) |
| `project` | Open-source or commercial projects |
| `framework` | Libraries, frameworks, platforms |
| `pattern` | Design patterns, best practices |
| `opinion` | Analysis, essays, editorial takes |
| `agent` | AI agents, autonomous systems |
| `knowledge` | Knowledge management, PKM, second brain |

---

## Wikilink Rules

- Every page must contain at least **2 outbound `[[wikilinks]]`** to other wiki pages
- Use the exact page title as the link: `[[LLM Wiki Pattern]]`
- Alias when needed: `[[LLM Wiki Pattern|Karpathy pattern]]`
- After ingest, run the cross-linker: scan all wiki pages and insert missing links where a page title appears as plain text

---

## Page Size

- Target: 50–150 lines of content (excluding frontmatter)
- Hard limit: **200 lines**
- Pages exceeding 200 lines must be split into sub-pages linked from the parent with a `## Subtopics` section

---

## Update Policy

When new information is ingested:

1. **New entity/concept not in wiki** → create a page
2. **Entity/concept already exists** → update the existing page; add new source to `sources:` list; update `updated:` date
3. **New info contradicts existing** → add both positions with date/source attribution; set `contradictions: true`
4. **New info duplicates existing** → skip; do not create a duplicate page

---

## Raw Sources — Immutability Rule

Files in `raw/` are **never modified** after creation. All corrections, annotations, and synthesis go into wiki pages. The `raw/` file is the permanent record of the source as ingested.

---

## Index Format (`index.md`)

```markdown
| Page | Type | Tags | Summary |
|------|------|------|---------|
| [[Page Title]] | concept | ai, pattern | One-line summary |
```

One row per wiki page. Maintain alphabetical order within each type group.

---

## Log Format (`log.md`)

```markdown
## YYYY-MM-DD operation | Source Title

- Pages created: [[Page A]], [[Page B]]
- Pages updated: [[Page C]]
- Notes: any relevant detail
```

Operations: `ingest` | `convert` | `query` | `lint` | `setup`

Always append; never edit existing entries.

---

## Manifest Format (`.manifest.json`)

```json
{
  "raw/slug.md": {
    "converted_at": "YYYY-MM-DD",
    "original_path": "inbox/original.pdf",
    "raw_path": "raw/slug.md",
    "ingested_at": "YYYY-MM-DD",
    "wiki_pages": ["wiki/concepts/slug.md", "wiki/entities/author.md"]
  }
}
```

Used by `wiki-convert` and `wiki-ingest` for delta processing — skip sources already processed unless the source file has changed.
