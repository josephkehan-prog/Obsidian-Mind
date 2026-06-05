# Obsidian Mind — LLM Wiki

This vault implements Andrej Karpathy's LLM Wiki pattern: instead of re-discovering knowledge per query via RAG, this system compiles sources once into structured, interlinked markdown pages and maintains them over time.

**You (the LLM) own the wiki layer. The human owns the raw sources.**

---

## Session Orientation (MANDATORY)

Before performing any operation on this vault, read these three files in order:

1. `SCHEMA.md` — conventions, page types, tag taxonomy, frontmatter rules
2. `index.md` — master catalog of all wiki pages (understand what exists)
3. `log.md` (last 20 entries) — recent operations (understand what changed recently)

This prevents duplicate pages, missed cross-references, and schema drift.

---

## Five Operations

### 1. `/wiki-setup`
Initialize or verify vault structure. Creates missing directories, bootstraps SCHEMA/index/log/manifest if absent.
See `.claude/skills/wiki-setup/SKILL.md`.

### 2. `/wiki-convert`
**Convert files from `inbox/` into clean markdown in `raw/`.**
Handles: PDF, DOCX, PPTX, ODT, EPUB, HTML, RTF, images (via vision).
Trigger phrases: "convert", "I dropped a file in inbox", file extension in message.
See `.claude/skills/wiki-convert/SKILL.md`.

### 3. `/wiki-ingest`
**Process raw markdown sources into structured wiki pages.**
4-stage pipeline: read → extract → merge → update index/log/manifest.
Trigger phrases: "ingest", "add to wiki", "process raw".
See `.claude/skills/wiki-ingest/SKILL.md`.

### 4. `/wiki-query`
**Answer questions using the wiki as a knowledge base.**
Tiered read — page titles/tags first, full bodies only when needed.
Files substantial answers as query pages.
Trigger: any question about topics that may be in the wiki.
See `.claude/skills/wiki-query/SKILL.md`.

### 5. `/wiki-lint`
**Health check the wiki.**
Finds: orphaned pages, broken wikilinks, missing frontmatter, stale sources, contradictions, oversized pages.
See `.claude/skills/wiki-lint/SKILL.md`.

---

## Key Conventions (quick reference)

- Wiki pages live in `wiki/{entities,concepts,comparisons,queries}/`
- Every page: YAML frontmatter + min 2 `[[wikilinks]]` + max 200 lines
- `raw/` files are immutable after creation
- `inbox/` is for binary originals awaiting conversion
- Always update `index.md` and `log.md` after ingest
- Tag only from the taxonomy in `SCHEMA.md`

---

## Workflow Summary

```
inbox/ ──(wiki-convert)──► raw/ ──(wiki-ingest)──► wiki/
                                                     │
                                              index.md + log.md
```

---

## Source Repositories

This vault was built on:
- [Karpathy LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [ar9av/obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- [NicholasSpisak/second-brain](https://github.com/NicholasSpisak/second-brain)
