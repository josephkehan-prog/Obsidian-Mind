# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Obsidian Mind — LLM Wiki

This vault implements Andrej Karpathy's LLM Wiki pattern: instead of re-discovering knowledge per query via RAG, this system compiles sources once into structured, interlinked markdown pages and maintains them over time.

**You (the LLM) own the wiki layer. The human owns the raw sources.**

This is a markdown vault, not a code project — there is no build, lint, or test toolchain. The "commands" are the five wiki skills below, plus the document converters listed under Conversion Tooling.

---

## Session Orientation (MANDATORY)

Before performing any operation on this vault, read these three files in order:

1. `SCHEMA.md` — conventions, page types, tag taxonomy, frontmatter rules
2. `index.md` — master catalog of all wiki pages (understand what exists)
3. `log.md` (last 20 entries) — recent operations (understand what changed recently)

This prevents duplicate pages, missed cross-references, and schema drift.

---

## Five Operations

| Skill | Purpose | Trigger phrases |
|---|---|---|
| `/wiki-setup` | Initialize/verify vault structure; bootstrap SCHEMA/index/log/manifest | new or damaged vault |
| `/wiki-convert` | Convert `inbox/` binaries (PDF, DOCX, PPTX, ODT, EPUB, HTML, RTF, images) → clean markdown in `raw/` | "convert", "I dropped a file in inbox", a file extension |
| `/wiki-ingest` | Process `raw/` markdown → structured wiki pages; 4 stages: read → extract → merge → update index/log/manifest | "ingest", "add to wiki", "process raw" |
| `/wiki-query` | Answer questions from the wiki; tiered read (titles/tags first, bodies only when needed); files substantial answers as query pages | any question the wiki may cover |
| `/wiki-lint` | Health check: orphans, broken wikilinks, missing frontmatter, stale sources, contradictions, oversized pages | periodic maintenance |

Each skill's full procedure lives in `.claude/skills/<name>/SKILL.md`.

---

## Data Flow

```
inbox/ ──(wiki-convert)──► raw/ ──(wiki-ingest)──► wiki/{entities,concepts,comparisons,queries}/
                                                     │
                                       index.md + log.md + .manifest.json updated
```

`output/` holds generated reports and exports.

### `.manifest.json` — delta tracking

The manifest is keyed by `raw/<slug>.md` and drives skip-if-already-processed logic:

- Inbox file with **no** manifest entry matching `original_path` → pending conversion
- Manifest entry with `"ingested_at": null` (or a `raw/*.md` with no entry) → pending ingestion
- Fully populated entry → skip unless the source changed (or user says `--force`)

---

## Key Rules (quick reference — SCHEMA.md is authoritative)

- Every wiki page: YAML frontmatter (`title`, `type`, `created`, `updated`, `tags`, `sources`, `contradictions`) + min 2 outbound `[[wikilinks]]` + max 200 lines (target 50–150)
- **Page-creation threshold:** content appears in 2+ sources, or is central in 1 source with ≥3 distinct facts. No stub pages.
- **Update, don't duplicate:** if a page exists, merge into it, append to `sources:`, bump `updated:`
- **Contradictions:** keep both positions with date/source attribution; set `contradictions: true`
- `raw/` files are **immutable** after creation — corrections go in wiki pages
- `log.md` is append-only; never edit existing entries
- Tags only from the taxonomy in `SCHEMA.md`; add new tags to SCHEMA.md first

---

## Conversion Tooling

`wiki-convert` probes for tools and uses what's available:

| Format | Primary | Fallback |
|---|---|---|
| PDF | `markitdown` | pymupdf script → `pdftotext` |
| DOCX / PPTX | `markitdown` | `pandoc -t markdown` |
| ODT / EPUB / HTML / RTF | `pandoc -t markdown` | `markitdown` |
| Images | Claude vision (Read tool) | `tesseract` |

Install: `pip install markitdown pymupdf`, `winget/brew/apt install pandoc`.

---

## Automation — wiki-daemon

`.claude/daemons/wiki-daemon/PROMPT.md` runs on a schedule (every 6h, registered as a CCR trigger). Each run: converts pending inbox files, ingests pending raw files, runs lint if the last lint is >7 days old, appends a `daemon | scheduled run` heartbeat to `log.md`, then commits and pushes.

When operating manually, check `log.md` for recent daemon entries — the daemon may have already converted/ingested what you're about to process. Keep daemon-relevant invariants intact: it relies on `.manifest.json` semantics and the `log.md` entry format above.

---

## Git & MCP

- The working branch is `master` (origin/main on GitHub is a stale initial commit).
- The Obsidian **obsidian-git** plugin and the daemon both commit; daemon commits use the `daemon: YYYY-MM-DD — summary` message format.
- `.mcp.json` configures an Obsidian Local REST API MCP server (`https://127.0.0.1:27124/mcp/`) for direct vault read/write through Obsidian. It contains a bearer token and must stay untracked — never `git add` it.

---

## Other Agents

`AGENTS.md` is the bootstrap for non-Claude agents (Hermes, Codex, Gemini CLI) and mirrors the rules above. Hermes integration uses `OBSIDIAN_VAULT_PATH` (see `.env.example`) and `.hermes/skills/obsidian-wiki/SKILL.md`. Keep AGENTS.md in sync when changing vault conventions.

---

## Source Repositories

This vault was built on:
- [Karpathy LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [ar9av/obsidian-wiki](https://github.com/ar9av/obsidian-wiki)
- [NicholasSpisak/second-brain](https://github.com/NicholasSpisak/second-brain)
