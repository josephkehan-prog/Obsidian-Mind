---
name: wiki-lint
description: Health-check the wiki. Finds orphaned pages, broken wikilinks, missing frontmatter fields, contradictions needing review, oversized pages, and stale sources. Run periodically to keep the wiki coherent.
---

# Wiki Lint

## When to Activate

- User runs `/wiki-lint`
- User says "check the wiki", "health check", "find broken links", "audit wiki"
- After a large batch ingest
- Periodically (suggested: monthly)

## Checks to Run

### 1. Broken Wikilinks

Scan every wiki page for `[[Link Text]]` patterns. For each link, check whether a file with the matching title exists in any `wiki/` subdirectory.

Report: `[[Broken Link]]` in `wiki/concepts/page.md`

### 2. Orphaned Pages

Build the full link graph. A page is orphaned if no other wiki page links to it (zero inbound links) AND it is not listed in `index.md`.

Report: `wiki/entities/orphan.md` — no inbound links

### 3. Missing or Invalid Frontmatter

For every wiki page, check:
- All required fields present: `title`, `type`, `created`, `updated`, `tags`, `sources`, `contradictions`
- `type` is one of: `entity`, `concept`, `comparison`, `query`
- `tags` contains only values from the SCHEMA taxonomy
- `sources` paths actually exist in `raw/`
- `created` and `updated` are valid ISO dates

Report each missing field individually.

### 4. Pages Missing Wikilinks

Check every wiki page has at least 2 outbound `[[wikilinks]]`.

Report: `wiki/concepts/page.md` — only 1 wikilink (needs ≥ 2)

### 5. Oversized Pages

Check every wiki page length. Flag any page exceeding 200 lines.

Report: `wiki/concepts/large-page.md` — 347 lines (split recommended)

### 6. Contradictions Needing Review

Find all wiki pages with `contradictions: true` in frontmatter.

List them so the user knows they need human review.

### 7. Index Consistency

- Every file in `wiki/` should have a row in `index.md`
- Every row in `index.md` should point to a file that exists
- Report mismatches in both directions

### 8. Stale Sources

Check `.manifest.json`. Find `raw/` entries where `ingested_at` is older than 90 days AND the raw file has been modified since `ingested_at`. These may need re-ingesting.

Report: `raw/old-paper.md` modified 2025-11-01, last ingested 2025-10-01 — may be stale

## Report Format

```
Wiki Lint Report — YYYY-MM-DD
══════════════════════════════

BROKEN WIKILINKS (N)
  wiki/concepts/transformers.md: [[Attentino Mechanism]] (typo?)
  wiki/entities/karpathy.md: [[OpenAI GPT-4]] (page doesn't exist)

ORPHANED PAGES (N)
  wiki/entities/unused-entity.md — 0 inbound links

MISSING FRONTMATTER (N)
  wiki/concepts/rag.md: missing 'sources' field
  wiki/entities/bert.md: invalid type 'model' (must be entity|concept|comparison|query)

PAGES WITHOUT ENOUGH WIKILINKS (N)
  wiki/comparisons/rag-vs-wiki.md — 1 wikilink (needs ≥ 2)

OVERSIZED PAGES (N)
  wiki/concepts/neural-networks.md — 312 lines (split at ## Architecture)

CONTRADICTIONS TO REVIEW (N)
  wiki/concepts/scaling-laws.md — marked contradictions: true

INDEX MISMATCHES (N)
  In wiki/ but not in index.md: wiki/entities/new-entity.md
  In index.md but file missing: [[Old Page]]

STALE SOURCES (N)
  raw/paper-2024.md — re-ingest recommended

──────────────────────────────
Total issues: N  |  Critical: N  |  Warnings: N
```

## After the Report

For each finding, ask the user if they want to:
- Fix it now (Claude attempts the fix inline)
- Skip it
- Mark it as won't-fix

Fixable automatically:
- Typo wikilinks (suggest correction)
- Missing index rows (add them)
- Missing frontmatter fields with obvious values (fill in)

Requires human judgment:
- Contradictions
- Orphaned pages (decide: delete or link)
- Oversized pages (decide: where to split)
- Stale sources (decide: re-ingest or ignore)

## Log Entry

Append to `log.md`:
```markdown
## YYYY-MM-DD lint | Full Audit

- Issues found: N (N critical, N warnings)
- Fixed automatically: [list]
- Deferred to user: [list]
```

## Examples

```
/wiki-lint
health check the wiki
find broken links
audit the vault
check for orphaned pages
```
