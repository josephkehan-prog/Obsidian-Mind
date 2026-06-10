# Obsidian ↔ Claude Workflows — Research Report

Date: 2026-06-10 · Method: Exa neural search (3 queries, ~24 sources) · Purpose: survey workflows beyond this vault's current inbox→raw→wiki pipeline, focused on getting markdown found, captured, and filed automatically.

---

## 1. Capture pipelines — getting markdown INTO the vault

These fill the front of the pipeline (this vault's `inbox/` and `raw/`). Currently the only way content arrives here is manual file drops.

| Workflow | How it works | Fit for this vault |
|---|---|---|
| **Obsidian Web Clipper** (official) | Browser extension; one click saves article/tweet/YouTube (with timestamped transcript) as markdown to a chosen folder | **High** — point it at `inbox/`; the 6h daemon converts + ingests automatically |
| **Ingester plugin** (community) | Watches a folder; on new file, runs `claude "/ingest <path>"` in tmux — built exactly for the Karpathy wiki pattern | Medium — instant ingest vs 6h daemon, but requires tmux/zsh (macOS/Linux; not native on Windows) |
| **mail2note** | Forward email to a personal `@mail2note.com` address → clean markdown + YAML frontmatter + attachments synced into a vault folder via plugin | High for email capture — newsletters, meeting notes |
| **Readwise official sync** | Highlights from articles/books/tweets sync to a `Readwise/` folder as atomic notes | High if Readwise user |
| **Readwise → n8n → vault** | n8n polls Readwise Export API, formats markdown in a Code node, writes via Local REST API (`PATCH`/append for existing notes) | Only if custom routing/AI-summarization needed pre-vault |
| **n8n multi-source routing** | Telegram bot for mobile ideas, Whisper for voice notes, Readwise for reading — n8n formats and drops markdown into the vault; scheduled Claude "daily briefing" reads the last 24h | Medium — most capture breadth, most moving parts |
| **Claude Sync plugin** | Watches `~/Downloads/Claude Exports`, auto-imports Claude chat exports into the vault preserving structure | Low-effort way to capture Claude conversations as sources |

## 2. Full LLM-wiki automation (same pattern as this vault)

- **fakechris/obsidian_vault_pipeline** — production-grade Karpathy-pattern implementation: directory watcher ("AutoPilot"), LLM quality scoring with auto-retry, evergreen-note extraction, MOC index maintenance, broken-link auto-repair, auto-commit. CLI: `ovp-autopilot --watch=inbox`. Worth mining for ideas: **quality gates** (line counts, placeholder detection, frontmatter validation as pre-commit checks) and **LLM quality scoring with retry below threshold**.
- **Codex knowledge wiki tutorial** (marketingagent.blog) — same architecture with OpenAI Codex: `agents.md` ingest sub-prompt, Web Clipper → `raw/`, nightly scheduled automation ("if unprocessed files exist, process them"). Validates this vault's daemon design.

## 3. Operating workflows (Claude working ON the vault)

- **Daily/weekly loops** (multiple guides converge on this): `/today` builds the day's note carrying over unfinished tasks; `/log` structures an evening journal entry; `/sunday`/weekly-review reads the week's daily notes and surfaces patterns. Implemented as slash commands (markdown files in `.claude/commands/`).
- **Meeting notes → action items**: read unstructured daily note, extract commitments into a checkbox task note linked back to source.
- **eferro's reliability pattern** (most engineering-minded writeup): reliability comes from explicit conventions, not the model — `.claude/rules/` files as operational memory, plus **Python verification scripts** (`find_broken_links.py`, `fix_broken_links.py` with difflib fuzzy match + `--auto-apply --threshold 0.90`, `validate_frontmatter.py`) that verify what the agent thinks it did. Skills turn vague instructions into written protocols.
- **Bounded permissions** (claudecode-lab): per-folder safety map in `.claude/settings.json` — e.g. `inbox/` create+triage, `archive/` no edits, `private/` no reads. Their finding: narrow scopes + link audits worked; "clean up the whole vault" prompts invented tags and headings.

## 4. MCP server landscape (this vault already has Local REST API on :27124)

| Server | Distinguishing features |
|---|---|
| `mcp-obsidian` (MarkusPfundstein) | The de-facto standard; 7 tools over Local REST API; `uvx mcp-obsidian` |
| `swarogan/mcp-obsidian` | 18 tools; PATCH v3 (heading/block/frontmatter targeting, search-replace); Templater execution; web fetch→markdown |
| `jagoff/obsidian-mcp` | 66 tools, **local-first — works without Obsidian running, no plugin**; BM25 search, graph traversal, canvas, batch ops |
| `smart-connections-mcp` / `optimike-obsidian-mcp` | **Semantic search** over Smart Connections embeddings; similar-note discovery, connection graphs |
| `@connorbritain/obsidian-mcp-server` | Graph tools: orphan detection, PageRank, cluster detection (overlaps `/wiki-lint`) |

Gotcha repeatedly flagged: `obsidian-mcp` (npx, direct file access) ≠ `mcp-obsidian` (uvx, REST API) — different packages.

## 5. Recommendations for Obsidian Mind

1. **Install Obsidian Web Clipper → save to `inbox/`** — highest-leverage, zero-code; the existing daemon picks clips up within 6h. This unblocks "finding and placing markdowns" today.
2. **Add an email lane** (mail2note, or Gmail MCP already connected in Claude Code) for newsletters/meeting notes.
3. **Borrow the verification-script pattern** from eferro/ovp: a deterministic `validate_frontmatter` / broken-link script the daemon's lint step can run, instead of pure LLM judgment.
4. **Later, once the wiki has content**: Smart Connections + an MCP semantic layer for similarity-based cross-linking during ingest.

## Sources

- https://dev.to/mibii/claude-code-obsidian-build-a-second-brain-that-actually-thinks-d61
- https://www.codewithseb.com/blog/claude-code-obsidian-second-brain-guide
- https://eferro.substack.com/p/how-i-use-claude-code-to-maintain
- https://markanamedia.com/blog/claude-code-obsidian/
- https://dev.to/malik_chohra/how-to-build-a-second-brain-with-obsidian-and-claude-code-step-by-step-5gol
- https://claudecode-lab.com/en/blog/claude-code-obsidian-integration/
- https://frankanaya.com/claude-code/
- https://mcp.directory/blog/obsidian-mcp-complete-guide-2026
- https://community.obsidian.md/plugins/ingester
- https://community.obsidian.md/plugins/claude-sync
- https://ai.plainenglish.io/the-automated-obsidian-intelligence-vault-that-gets-smarter-every-day-709e240150d3
- https://notes-automate.com/posts/extracting-readwise-highlights-to-obsidian-via-n8n/
- https://mail2note.com/
- https://marketingagent.blog/2026/05/05/tutorial-obsidian-openai-codex-personal-knowledge-wiki/
- https://github.com/fakechris/obsidian_vault_pipeline
- https://github.com/jagoff/obsidian-mcp · https://github.com/swarogan/mcp-obsidian · https://github.com/punkpeye/obsidian-mcp · https://github.com/msdanyg/smart-connections-mcp · https://www.npmjs.com/package/@connorbritain/obsidian-mcp-server
