# Obsidian Mind — Multi-Agent Bootstrap

This file is the always-on context for non-Claude agents (Hermes, Codex, Gemini CLI, etc.) operating on this vault.

---

## Session Orientation (MANDATORY)

Before any operation, read in order:

1. `SCHEMA.md` — conventions, page types, tag taxonomy, frontmatter spec
2. `index.md` — master catalog of existing wiki pages
3. `log.md` (last 20 entries) — recent activity

Skipping this causes duplicate pages and broken cross-references.

---

## Vault Structure

```
inbox/       drop binary originals here (.pdf .docx .pptx .epub .png …)
raw/         converted, cleaned markdown — immutable after creation
wiki/
  entities/  people, tools, projects, organisations
  concepts/  ideas, frameworks, methodologies
  comparisons/ side-by-side analyses
  queries/   filed Q&A answers
output/      generated reports
index.md     master catalog
log.md       append-only operation log
SCHEMA.md    full conventions (read this first)
```

---

## Five Operations

| Command | What it does |
|---------|-------------|
| `wiki-setup` | Verify/initialize vault structure |
| `wiki-convert` | Convert inbox/ files → raw/ markdown |
| `wiki-ingest` | Process raw/ markdown → wiki pages + update index/log |
| `wiki-query` | Answer questions from wiki knowledge base |
| `wiki-lint` | Audit for broken links, missing frontmatter, orphans |

---

## Frontmatter Every Wiki Page Requires

```yaml
---
title: <title>
type: entity | concept | comparison | query
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
sources: [raw/slug.md]
contradictions: false
---
```

---

## Rules That Must Not Be Broken

- Min 2 outbound `[[wikilinks]]` per wiki page
- Max 200 lines per page (split if larger)
- Raw files in `raw/` are never modified
- Always append to `log.md`; never edit existing entries
- Only use tags from the taxonomy in `SCHEMA.md`

---

## Hermes Agent Integration (secondary)

Hermes ships built-in `obsidian` and `llm-wiki` skills that work with this vault out of the box.

**Setup:**
```bash
# Add to ~/.hermes/.env
OBSIDIAN_VAULT_PATH=/path/to/this/vault
```

Once set, Hermes' `obsidian` skill resolves all file paths relative to `OBSIDIAN_VAULT_PATH`. Use the `llm-wiki` skill for ingest/query operations. See `.hermes/skills/obsidian-wiki/SKILL.md` for vault-specific guidance.
