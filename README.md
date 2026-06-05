# Obsidian Mind

A personal knowledge base built on [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — an AI-maintained second brain that lives in Obsidian and is operated by Claude Code.

Instead of re-discovering knowledge from scratch on every query (RAG), this system compiles sources once into structured, interlinked markdown pages and keeps them current over time. The LLM does the bookkeeping; you do the thinking.

---

## How It Works

```
inbox/          ← drop .pdf .docx .pptx .epub .png here
    │
    ▼  /wiki-convert  (markitdown · pandoc · Claude vision)
raw/            ← cleaned markdown sources (immutable)
    │
    ▼  /wiki-ingest
wiki/
  entities/     ← people, tools, projects, organisations
  concepts/     ← ideas, frameworks, methodologies, patterns
  comparisons/  ← side-by-side analyses
  queries/      ← filed Q&A answers
    │
    ▼
index.md + log.md updated → obsidian-git commits
```

---

## Quick Start

### 1. Install converters (one-time)

```bash
pip install markitdown          # handles PDF, DOCX, PPTX, EPUB, HTML
brew install pandoc             # or: apt install pandoc
pip install pymupdf             # PDF fallback
```

### 2. Open the vault in Claude Code

```bash
claude /path/to/this/vault
```

`CLAUDE.md` at the vault root is auto-loaded — Claude knows the full wiki system immediately.

### 3. Add a document

Drop any file into `inbox/`, then:

```
/wiki-convert     → converts inbox/ files to raw/ markdown
/wiki-ingest      → builds wiki pages from raw/ sources
```

### 4. Query your knowledge

Just ask Claude a question. It reads `index.md` first (cheap), opens relevant pages only when needed, and cites wiki pages in its answer.

### 5. Keep it clean

```
/wiki-lint        → finds broken links, orphans, bad frontmatter, oversized pages
```

---

## Five Skills

| Command | What it does |
|---------|-------------|
| `/wiki-setup` | Verify vault structure, initialize missing files |
| `/wiki-convert` | Convert PDF/DOCX/PPTX/EPUB/images → clean markdown in `raw/` |
| `/wiki-ingest` | Process `raw/` sources → structured wiki pages with `[[wikilinks]]` |
| `/wiki-query` | Answer questions from the wiki knowledge base |
| `/wiki-lint` | Audit for broken links, orphans, missing frontmatter, contradictions |

Skills live in `.claude/skills/` and are loaded automatically by Claude Code.

---

## Document Conversion Support

| Format | Primary tool | Fallback |
|--------|-------------|---------|
| PDF | `markitdown` | `pymupdf` → `pdftotext` |
| DOCX / PPTX / ODT | `markitdown` | `pandoc` |
| EPUB | `pandoc` | `markitdown` |
| HTML | `pandoc --strip-comments` | inline strip |
| Images (OCR) | Claude vision | `tesseract` |
| TXT / RTF | direct copy | `pandoc` |

---

## Vault Structure

```
.
├── CLAUDE.md           Claude Code bootstrap (auto-loaded)
├── AGENTS.md           Multi-agent bootstrap (Hermes, Codex, etc.)
├── SCHEMA.md           Conventions: page types, frontmatter, tag taxonomy
├── index.md            Master catalog — one row per wiki page
├── log.md              Append-only operation log
├── .manifest.json      Delta tracker — skips already-processed sources
├── .env.example        OBSIDIAN_VAULT_PATH for Hermes integration
│
├── inbox/              Drop binary originals here
├── raw/                Converted markdown (LLM never modifies)
├── wiki/
│   ├── entities/
│   ├── concepts/
│   ├── comparisons/
│   └── queries/
├── output/             Generated reports and exports
│
└── .claude/skills/
    ├── wiki-setup/
    ├── wiki-convert/
    ├── wiki-ingest/
    ├── wiki-query/
    └── wiki-lint/
```

---

## Wiki Page Format

Every page the LLM creates follows this structure:

```yaml
---
title: Page Title
type: entity | concept | comparison | query
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [ai, tooling]
sources: [raw/source.md]
contradictions: false
---
```

- Minimum 2 outbound `[[wikilinks]]` per page
- Maximum 200 lines — split larger pages
- Tags from the taxonomy in `SCHEMA.md` only

---

## Hermes Agent (Secondary)

Hermes ships built-in `obsidian` and `llm-wiki` skills that work with this vault:

```bash
# ~/.hermes/.env
OBSIDIAN_VAULT_PATH=/path/to/this/vault
```

`AGENTS.md` at the vault root is picked up automatically. See `.hermes/skills/obsidian-wiki/SKILL.md` for vault-specific guidance.

---

## Based On

- [Andrej Karpathy — LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [ar9av/obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- [NicholasSpisak/second-brain](https://github.com/NicholasSpisak/second-brain)
- [NousResearch/hermes-agent](https://github.com/nousresearch/hermes-agent)
