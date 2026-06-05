---
name: wiki-convert
description: Convert binary documents (PDF, DOCX, PPTX, EPUB, HTML, images, etc.) from inbox/ into clean ingestable markdown in raw/. Run before wiki-ingest when sources are not already markdown.
---

# Wiki Convert

## When to Activate

- User runs `/wiki-convert`
- User says "convert", "I dropped a file in inbox", "process this PDF/doc"
- User mentions a file extension: `.pdf`, `.docx`, `.pptx`, `.odt`, `.epub`, `.html`, `.rtf`, `.png`, `.jpg`
- Files are present in `inbox/` and not yet in `.manifest.json`

## Tooling Check

Before converting, verify which tools are available. Run these checks and report what's missing:

```bash
# Check markitdown (handles PDF, DOCX, PPTX, EPUB, HTML, images)
python -c "import markitdown" 2>/dev/null && echo "markitdown: ok" || echo "markitdown: missing — pip install markitdown"

# Check pandoc (handles DOCX, EPUB, HTML, RTF, ODT)
pandoc --version 2>/dev/null | head -1 || echo "pandoc: missing — brew install pandoc / apt install pandoc"

# Check pymupdf (PDF fallback)
python -c "import fitz" 2>/dev/null && echo "pymupdf: ok" || echo "pymupdf: missing — pip install pymupdf"

# Check pdftotext (PDF last resort)
pdftotext -v 2>/dev/null | head -1 || echo "pdftotext: missing — apt install poppler-utils / brew install poppler"
```

Proceed with whatever is available. If nothing can handle the detected format, tell the user exactly what to install.

## Conversion Matrix

| Extension | Primary converter | Fallback |
|-----------|------------------|---------|
| `.pdf` | `markitdown <file>` | pymupdf script → `pdftotext` |
| `.docx` | `markitdown <file>` | `pandoc -t markdown` |
| `.pptx` | `markitdown <file>` | `pandoc -t markdown` |
| `.odt` | `pandoc -t markdown` | `markitdown <file>` |
| `.epub` | `pandoc -t markdown --strip-comments` | `markitdown <file>` |
| `.html` / `.htm` | `pandoc -t markdown --strip-comments` | strip tags + markitdown |
| `.rtf` | `pandoc -t markdown` | — |
| `.txt` | direct copy, normalize | — |
| `.png` / `.jpg` / `.jpeg` / `.webp` | Claude vision (Read tool) → describe as markdown | `tesseract` |

## Conversion Steps

### Step 1: Discover files

List all files in `inbox/` that are not already in `.manifest.json` under `original_path`. These are the pending conversions.

If no pending files, tell the user inbox is empty or all files are already converted.

### Step 2: For each pending file

**A. Detect format** by file extension (case-insensitive).

**B. Run primary converter:**

```bash
# markitdown (covers most formats)
markitdown inbox/filename.pdf > /tmp/converted_raw.md

# pandoc
pandoc inbox/filename.epub -t markdown --strip-comments -o /tmp/converted_raw.md
```

For images: use the Read tool to view the image, then write a markdown description capturing all text, diagrams, tables, and figures as structured markdown.

**C. Post-clean the output:**

Apply these cleaning rules to the converted markdown:

1. Remove page number lines — patterns like `Page N of M`, `- N -`, lone integers on their own line
2. Remove repeated running headers/footers — detect 3+ identical lines across the document and strip them
3. Collapse 3+ consecutive blank lines → 2 blank lines
4. Normalize bullet styles — convert `•`, `·`, `*` bullets to `-`
5. Remove watermark text — patterns like `CONFIDENTIAL`, `DRAFT`, `DO NOT DISTRIBUTE` on their own line (preserve if embedded in a sentence)
6. Fix broken hyphenation — re-join words split across lines with `word-↵word` patterns
7. Normalize heading levels — ensure h1 is used only once (document title), shift levels if needed

**D. Infer slug filename:**
- Start with the original filename, strip extension
- Convert to lowercase snake_case: `My Research Paper 2024.pdf` → `my_research_paper_2024`
- If the filename is ambiguous (e.g., `document1`, `untitled`), prefix with today's date: `2026-06-05_document1`
- Output path: `raw/<slug>.md`

**E. Write cleaned markdown to `raw/<slug>.md`**

**F. Update `.manifest.json`:**
```json
{
  "raw/slug.md": {
    "converted_at": "YYYY-MM-DD",
    "original_path": "inbox/original.pdf",
    "raw_path": "raw/slug.md",
    "ingested_at": null,
    "wiki_pages": []
  }
}
```

### Step 3: Report and log

Print a conversion summary:
```
Wiki Convert Complete
─────────────────────
Converted:  N files
  ✓ inbox/paper.pdf  →  raw/paper.md  (markitdown, 847 lines)
  ✓ inbox/notes.docx →  raw/notes.md  (pandoc, 124 lines)
  ✗ inbox/image.png  →  raw/image_description.md  (claude vision)

Ready to ingest: run /wiki-ingest
```

Append to `log.md`:
```markdown
## YYYY-MM-DD convert | <filenames>

- Files converted: inbox/paper.pdf → raw/paper.md, ...
- Tools used: markitdown, pandoc
- Notes: [any issues]
```

## Edge Cases

- **Password-protected PDF:** report it, skip it, tell user to unlock first
- **Scanned PDF (no text layer):** fall back to Claude vision if page count ≤ 20; warn user for longer docs
- **File already in manifest:** skip unless `--force` is mentioned
- **Conversion produces <50 words:** warn the user the output looks suspiciously short; show first 10 lines for verification

## Examples

```
/wiki-convert
convert the pdf I dropped in inbox
process all inbox files
convert notes.docx
```
