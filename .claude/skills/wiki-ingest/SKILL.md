---
name: wiki-ingest
description: Process raw markdown sources into structured wiki pages with wikilinks and frontmatter. Updates index.md, log.md, and .manifest.json. Run after wiki-convert or when new .md files appear in raw/.
---

# Wiki Ingest

## When to Activate

- User runs `/wiki-ingest`
- User says "ingest", "add to wiki", "process raw", "update the wiki"
- New `.md` files appear in `raw/` without an `ingested_at` entry in `.manifest.json`

## Pre-Flight

1. Read `SCHEMA.md` — refresh conventions before processing
2. Read `index.md` — know what pages already exist (avoid duplicates)
3. Read `log.md` last 10 entries — understand recent activity

## Step 1: Identify Pending Sources

Open `.manifest.json`. Find all entries where `ingested_at` is `null`.

Also scan `raw/` for any `.md` files not in the manifest at all — add them with `converted_at: null, ingested_at: null`.

If nothing is pending, report "No new sources to ingest" and stop.

## Step 2: For Each Pending Source

Work through pending sources one at a time.

### 2a. Read and Analyse

Read the full raw markdown file. Extract:

**Entities** (named things): people, tools, projects, companies, papers, events
**Concepts** (ideas): frameworks, methodologies, techniques, patterns, theories
**Relationships**: how entities and concepts relate to each other
**Key claims**: the 5–10 most important factual statements
**Contradictions**: claims that may conflict with existing wiki pages

### 2b. Decide What to Create or Update

For each extracted entity/concept:

1. Check `index.md` — does a page already exist?
   - **Yes** → update the existing page (add new facts, update `updated:` date, add source to `sources:`)
   - **No** → create a new page if it meets the threshold (2+ sources or central to this source)

Use these mappings:
- Named person / tool / project / company → `wiki/entities/<slug>.md`
- Idea / method / framework / technique → `wiki/concepts/<slug>.md`
- Comparing 2+ things head-to-head → `wiki/comparisons/<slug>.md`

### 2c. Write or Update Each Wiki Page

**Page filename:** lowercase hyphenated slug of the title: `Transformer Architecture` → `transformer-architecture.md`

**Page structure:**

```markdown
---
title: <title>
type: entity | concept | comparison | query
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
sources: [raw/slug.md]
contradictions: false
---

# <title>

<One-sentence definition or summary.>

## Overview

<2–4 paragraphs of synthesised knowledge from all sources. Write in encyclopedic style — clear, factual, no filler.>

## Key Points

- Point one with [[wikilink]] where relevant
- Point two
- Point three (cite source if a specific claim)

## Related

- [[Related Page 1]] — brief relationship note
- [[Related Page 2]] — brief relationship note

## Sources

- `raw/slug.md` — [one-line description of the source]
```

**Wikilink rules:**
- Insert `[[Page Title]]` wherever another wiki page title is mentioned in the body
- Every page must have ≥ 2 outbound wikilinks
- If a relevant page doesn't exist yet, create it as a stub or queue it — don't omit the link

### 2d. Cross-Link Pass

After writing all new/updated pages for this source, scan every new page's body for mentions of other existing wiki page titles (from `index.md`) and insert the `[[wikilinks]]` if they're missing.

### 2e. Update index.md

For each new page, append a row to the index table:
```markdown
| [[Page Title]] | concept | ai, pattern | One-line summary of the page |
```

Keep rows alphabetical within type groups. Do not re-sort existing rows — append to the correct position.

### 2f. Update .manifest.json

```json
{
  "raw/slug.md": {
    "converted_at": "YYYY-MM-DD",
    "original_path": "inbox/original.pdf",
    "raw_path": "raw/slug.md",
    "ingested_at": "YYYY-MM-DD",
    "wiki_pages": ["wiki/concepts/page-a.md", "wiki/entities/page-b.md"]
  }
}
```

## Step 3: Report and Log

Print an ingest summary:
```
Wiki Ingest Complete
────────────────────
Source:  raw/paper.md
Created: [[Transformer Architecture]], [[Attention Mechanism]]
Updated: [[Neural Network]]
Skipped: 1 entity below threshold

Total wiki pages:  N entities, N concepts, N comparisons, N queries
```

Append to `log.md`:
```markdown
## YYYY-MM-DD ingest | <source title>

- Source: raw/slug.md
- Pages created: [[Page A]], [[Page B]]
- Pages updated: [[Page C]]
- Notes: [contradictions found, skipped items, etc.]
```

## Quality Rules

1. Never copy sentences verbatim from raw sources — synthesise and rephrase
2. Every claim should be traceable to a source (add inline citation if it's a specific fact)
3. If a claim contradicts an existing wiki page, note both positions inline and set `contradictions: true`
4. Do not create pages for trivial mentions — the entity/concept must have real substance
5. Tags must come from `SCHEMA.md` taxonomy only

## Examples

```
/wiki-ingest
ingest raw/karpathy_llm_wiki.md
add to wiki
process all new raw files
update the wiki with everything in raw/
```
