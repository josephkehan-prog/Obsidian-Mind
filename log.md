# Operation Log

Append-only chronological record of all wiki operations.

Format per entry:
```
## YYYY-MM-DD operation | Source Title
- Pages created: [[Page A]], [[Page B]]
- Pages updated: [[Page C]]
- Notes: detail
```

Operations: `setup` | `convert` | `ingest` | `query` | `lint`

---

<!-- Entries are appended below. Never edit or delete existing entries. -->

## 2026-06-05 setup | Vault Initialization

- Directories verified: inbox, raw, wiki/{entities,concepts,comparisons,queries}, output
- Files verified: CLAUDE.md, AGENTS.md, SCHEMA.md, index.md, log.md, .manifest.json, .env.example
- Skills verified: wiki-setup, wiki-convert, wiki-ingest, wiki-query, wiki-lint
- Hermes adapter: .hermes/skills/obsidian-wiki/SKILL.md
- Status: clean — 0 inbox, 0 raw, 0 wiki pages
