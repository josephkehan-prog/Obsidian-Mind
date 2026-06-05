---
name: obsidian-wiki
description: Adapter skill for Hermes agent to operate on this Obsidian LLM Wiki vault. Bridges vault conventions to Hermes' native obsidian and llm-wiki built-in skills.
---

# Obsidian Wiki (Hermes Adapter)

This skill is a thin adapter. Hermes' built-in `obsidian` and `llm-wiki` skills do the heavy work — this file tells Hermes about the vault-specific conventions it needs to follow.

## Setup

```bash
# Add to ~/.hermes/.env
OBSIDIAN_VAULT_PATH=/absolute/path/to/your/vault
```

Once set, Hermes' `obsidian` skill resolves all file paths relative to `OBSIDIAN_VAULT_PATH`. The `llm-wiki` skill implements Karpathy's 3-layer pattern and will work with this vault's structure.

## Session Orientation (MANDATORY)

Before any vault operation, read in order:

1. `$OBSIDIAN_VAULT_PATH/SCHEMA.md` — full conventions
2. `$OBSIDIAN_VAULT_PATH/index.md` — existing wiki pages
3. `$OBSIDIAN_VAULT_PATH/log.md` (last 20 lines) — recent activity

Use `read_file` with the absolute resolved path — file tools do not expand shell variables.

## Vault Structure

```
$OBSIDIAN_VAULT_PATH/
  inbox/          binary originals (.pdf .docx etc.) — do not modify
  raw/            converted markdown sources — do not modify
  wiki/entities/  create/update entity pages here
  wiki/concepts/  create/update concept pages here
  wiki/comparisons/ create/update comparison pages here
  wiki/queries/   create/update query pages here
  output/         generated reports
  index.md        append rows here after ingest
  log.md          append entries here after any operation
  .manifest.json  read/update delta tracking
```

## File Operations via Hermes Tools

| Operation | Tool | Notes |
|-----------|------|-------|
| Read a wiki page | `read_file` | Pass absolute path |
| List all wiki pages | `search_files` with `pattern: "*.md"` in `wiki/` | |
| Search page contents | `search_files` with content search | |
| Create a new wiki page | `write_file` | Full markdown with frontmatter |
| Update a section | `patch` | Use stable context anchors |
| Append to log.md | `patch` | Anchor at bottom |
| Update index.md | `patch` | Append new row to table |
| Update manifest | `read_file` → edit JSON → `write_file` | |

## Conventions to Follow

These match `SCHEMA.md` exactly — do not deviate:

**Page frontmatter (required):**
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

**Wikilink syntax:** `[[Page Title]]` — Obsidian wikilinks, not markdown links.

**Min 2 outbound wikilinks per page.**

**Page size limit: 200 lines.**

**Raw files in `raw/` are immutable** — Hermes must not write to them.

## Using Built-in Skills

The `llm-wiki` skill maps directly to this vault's operations:

| llm-wiki operation | This vault's equivalent |
|-------------------|------------------------|
| `ingest` | `wiki-ingest` (process `raw/` → `wiki/`) |
| `query` | `wiki-query` (tiered read from `wiki/`) |
| `lint` | `wiki-lint` (health check) |

When Hermes runs `llm-wiki ingest`, it should write to this vault's `wiki/` directories, update `index.md`, append to `log.md`, and update `.manifest.json`.

## Limitations

Hermes cannot run `/wiki-convert` directly (that requires CLI tools like markitdown/pandoc). For binary document conversion, use Claude Code's `wiki-convert` skill. Hermes can ingest already-converted `raw/*.md` files.
