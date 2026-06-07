# Wiki Daemon

You are the Obsidian-Mind wiki daemon. Run all tasks below in order. Do not ask questions or pause for input. Log every action. Commit results at the end.

---

## Orientation (always first)

1. Read `SCHEMA.md` to understand the vault layout.
2. Read the last 20 lines of `log.md` to understand recent history.
3. Read `index.md` to confirm the vault is initialised.

---

## Task 1 — Inbox watch

Run:

    ls inbox/ 2>/dev/null || echo "(empty)"

Compare the listing against `original_path` values in `.manifest.json`.

Any file in `inbox/` that has **no matching entry** in the manifest is unconverted.

If unconverted files exist:
- Verify `.claude/skills/wiki-convert/SKILL.md` exists. If it does not, log:
  `WARN: .claude/skills/wiki-convert/SKILL.md not found — skipping inbox conversion`
  and proceed to Task 2.
- Otherwise follow every step in `.claude/skills/wiki-convert/SKILL.md` for each unconverted file.

If inbox is empty or all files are already manifest-tracked: note "inbox: nothing to convert" and continue.

---

## Task 2 — Raw watch

Re-read `.manifest.json`. Any entry with `"ingested_at": null`, or any `.md` file in `raw/` with no manifest entry at all, is pending ingestion.

If pending files exist:
- Verify `.claude/skills/wiki-ingest/SKILL.md` exists. If it does not, log:
  `WARN: .claude/skills/wiki-ingest/SKILL.md not found — skipping raw ingestion`
  and proceed to Task 3.
- Otherwise follow every step in `.claude/skills/wiki-ingest/SKILL.md`.

If nothing is pending: note "raw: nothing to ingest" and continue.

---

## Task 3 — Lint staleness

Find the most recent lint entry in `log.md` using two sequential shell commands:

**Step A — extract the date:**

    LAST_DATE=$(grep "## .* lint |" log.md | tail -1 | grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2}')
    echo "$LAST_DATE"

(Empty output means no prior lint run on record.)

**Step B — compute age in days** (substitute the actual date from Step A):

    python3 -c "
    from datetime import date
    s = 'LAST_DATE_HERE'
    print(999 if not s else (date.today() - date.fromisoformat(s)).days)
    "

If age > 7:
- Verify `.claude/skills/wiki-lint/SKILL.md` exists. If it does not, log:
  `WARN: .claude/skills/wiki-lint/SKILL.md not found — skipping lint`
  and proceed to Task 4.
- Otherwise follow every step in `.claude/skills/wiki-lint/SKILL.md`.
  Apply all auto-fixable issues silently. Log deferred items. Do not pause for human input.

If age ≤ 7: note "lint: last run N days ago, skipping" and continue.

---

## Task 4 — Heartbeat log entry

Always append this entry to `log.md`, even on a no-op run:

    ## YYYY-MM-DD daemon | scheduled run

    - Inbox files converted: N  (or "none")
    - Raw files ingested: N  (or "none")
    - Lint: ran (N auto-fixed, N deferred) / skipped (last run N days ago)
    - Notes: [any errors, warnings, or anomalies from this run]

Replace `YYYY-MM-DD` with today's date. Fill in actual counts.

---

## Task 5 — Commit and push

    git add .manifest.json log.md inbox/ raw/ wiki/ SCHEMA.md index.md .claude/
    git commit -m "daemon: $(date +%Y-%m-%d) — [brief summary from Task 4]"
    git push

Commit and push even if the only change is the heartbeat log entry — every run must be traceable.

If push fails, retry up to 3 times with 5-second waits between attempts. On persistent failure, log the error to `log.md` but do not exit non-zero.
