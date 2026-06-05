---
name: wiki-query
description: Answer questions using the wiki as a knowledge base. Uses tiered reading (titles+tags first, bodies only when needed) to keep costs low. Files substantial answers as query pages for future reuse.
---

# Wiki Query

## When to Activate

- User asks a question that may be answerable from the wiki
- User runs `/wiki-query`
- User says "what does the wiki say about", "search the wiki for", "find in wiki"
- Any knowledge question after a vault has been populated

## Step 1: Session Orientation

Read `SCHEMA.md` and `index.md` before answering. This takes seconds and prevents fabricating answers about topics that don't exist in the wiki.

## Step 2: Tiered Reading

### Tier 1 — Cheap Pass (always run first)

Scan `index.md`. Look for pages whose title, type, or tags are relevant to the question.

If the cheap pass gives a clear answer → respond directly with inline citations (e.g. `per [[Transformer Architecture]]`).

### Tier 2 — Open Relevant Pages

If Tier 1 doesn't resolve the question, open the bodies of the 1–5 most relevant pages identified in Tier 1.

Read only what's needed. Synthesise across pages.

### Tier 3 — Wider Search

If the answer requires more context, open additional pages linked from the Tier 2 pages via `[[wikilinks]]`. Do not open more than 10 pages total without telling the user what you're doing.

## Step 3: Compose the Answer

Write the answer clearly:
- Cite specific wiki pages inline: `According to [[Attention Mechanism]]...`
- If information is absent from the wiki, say so explicitly: "This topic has not been ingested yet. You may want to run /wiki-ingest on a relevant source."
- Distinguish wiki knowledge (compiled facts) from your own reasoning (label as "My inference:")

## Step 4: Decide Whether to File the Answer

File the answer as a query page **when**:
- The answer is substantive (>3 paragraphs or synthesises ≥ 3 wiki pages)
- The question is likely to recur
- The synthesis reveals a non-obvious connection worth preserving

**Query page structure:**

```markdown
---
title: <the question, title-cased>
type: query
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [relevant, tags]
sources: [wiki/concepts/page-a.md, wiki/entities/page-b.md]
contradictions: false
---

# <Question>

## Answer

<synthesised answer with wikilinks>

## Related Pages

- [[Page A]]
- [[Page B]]
```

File to: `wiki/queries/<slug>.md`

Add a row to `index.md` and an entry to `log.md`:
```markdown
## YYYY-MM-DD query | <question title>

- Filed as: [[Query Title]]
- Pages consulted: [[Page A]], [[Page B]]
```

## When the Wiki Doesn't Know

If the topic is genuinely absent from the wiki:

1. Say so clearly
2. Suggest: "Run `/wiki-convert` on relevant sources then `/wiki-ingest` to build this knowledge"
3. Do not fabricate wiki content or pretend the wiki says something it doesn't

## Examples

```
/wiki-query
What does the wiki know about transformer architectures?
Search the wiki for Karpathy
What's the connection between attention and transformers?
Has anything been ingested about RAG?
```
