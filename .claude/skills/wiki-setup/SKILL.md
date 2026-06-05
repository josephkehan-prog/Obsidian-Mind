---
name: wiki-setup
description: Initialize or verify the Obsidian LLM Wiki vault structure. Creates missing directories, bootstraps SCHEMA/index/log/manifest if absent. Run once on a new vault or to repair a damaged one.
---

# Wiki Setup

## When to Activate

- User runs `/wiki-setup`
- User says "set up the wiki", "initialize vault", "something's missing"
- First time using a vault — run this before any other operation
- A required file or directory is missing

## Steps

### 1. Check Required Directories

Verify these directories exist. Create any that are missing:

```
inbox/
raw/
wiki/entities/
wiki/concepts/
wiki/comparisons/
wiki/queries/
output/
.claude/skills/
```

### 2. Check Required Root Files

For each file, if it is missing, create it with the canonical template below:

**`SCHEMA.md`** — if missing, tell the user it's the single most important file and they should restore it from git or the source repo.

**`index.md`** — if missing, create:
```markdown
# Wiki Index

| Page | Type | Tags | Summary |
|------|------|------|---------|
```

**`log.md`** — if missing, create:
```markdown
# Operation Log

Append-only chronological record of all wiki operations.
```

**`.manifest.json`** — if missing, create:
```json
{}
```

### 3. Validate SCHEMA.md

If SCHEMA.md exists, check it contains these sections:
- Directory Layout
- Page Types
- Required Frontmatter
- Tag Taxonomy
- Wikilink Rules

Report any missing sections.

### 4. Report

Print a setup summary:
```
Wiki Setup Complete
──────────────────
Directories:  ✓ all present
index.md:     ✓ / ⚠ created
log.md:       ✓ / ⚠ created
.manifest.json: ✓ / ⚠ created
SCHEMA.md:    ✓ / ✗ MISSING — restore required

Wiki pages:   N entities, N concepts, N comparisons, N queries
Raw sources:  N files
Inbox items:  N files awaiting conversion
```

### 5. Log the Operation

Append to `log.md`:
```markdown
## YYYY-MM-DD setup | Vault Initialization

- Directories verified/created
- Files verified/created: [list]
- Notes: [any issues found]
```

## Examples

```
/wiki-setup
set up the wiki
initialize vault
```
