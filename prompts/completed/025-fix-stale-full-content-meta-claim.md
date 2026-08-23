---
status: completed
summary: Corrected the stale 'Full content in meta' claim in CLAUDE.md to accurately state that self.meta stores only the resolved path per entry, matching the actual indexer behavior
execution_id: semantic-search-exec-025-fix-stale-full-content-meta-claim
dark-factory-version: dev
created: "2026-08-23T00:00:00Z"
queued: "2026-08-23T09:19:25Z"
started: "2026-08-23T09:19:45Z"
completed: "2026-08-23T09:21:00Z"
---

# Correct stale "Full content in meta" claim in CLAUDE.md

<summary>
- Fixes a wrong statement in the project's CLAUDE.md about what the index stores per entry
- The corrected claim matches the actual source: the index stores only the file path per entry
- No source code, tests, or runtime behavior change
- No user-facing docs (README, CHANGELOG) touched — the corrected claim is internal dev documentation only
- The change is confined to a single bullet in the Key Design Decisions section of CLAUDE.md
</summary>

<objective>
Correct the stale "Full content in meta" design-decision claim in CLAUDE.md so the documented behavior matches what the index actually stores (path only), preventing future developers from designing around a feature that does not exist.
</objective>

<context>
Read these files before making changes:

- `CLAUDE.md` — the stale claim is the bullet `- **Full content in meta** — \`self.meta\` stores entire file content per entry (for future features)` under `## Key Design Decisions`.
- `src/semantic_search/indexer.py` — confirm the true behavior before editing. `VaultIndexer.__init__` declares `self.meta: dict[str, dict[str, str]] = {}  # {idx: {"path": ...}}`; both `add_file_to_index` (writes `self.meta[str(new_idx)] = {"path": path_str}`) and `rebuild_index` (writes `new_meta[str(idx)] = {"path": str(file_path)}`) store only the resolved path per entry; `search` returns `{"path": ..., "score": ...}` with no content. Content is read from disk on demand in `get_content`, never stored in `self.meta`. The path-only storage dates to commit `6aafb0a` "002-fix-index-memory-leak" (2026-04-03).

Per `docs/dod.md`, a CHANGELOG entry is required only for user-facing changes; this internal dev-doc fix is not user-facing, so no CHANGELOG or README update.
</context>

<requirements>
1. In `CLAUDE.md`, under `## Key Design Decisions`, replace the bullet
   `- **Full content in meta** — \`self.meta\` stores entire file content per entry (for future features)`
   with wording that accurately states that `self.meta` stores only the resolved path per entry and that file content is never held in the index. Use this exact replacement (matches the section's terse `- **Name** — description` style):
   `- **Path-only meta** — \`self.meta\` stores only \`{"path": ...}\` per entry; file content is never kept in the index (path-only since the 2026-04-03 memory-leak fix)`
2. If the exact target bullet text no longer matches (edited by a prior change), correct whichever bullet in `## Key Design Decisions` still makes the false "full content in meta" claim, using the same path-only wording, and note the discrepancy in the report.
3. Make no other edits. Do not touch `README.md`, `CHANGELOG.md`, or anything under `src/` or `tests/` — this is a docs-only correction.
</requirements>

<constraints>
- Do NOT commit — dark-factory handles git
- Do not modify Python source, tests, README, or CHANGELOG
- Make no changes to CLAUDE.md beyond the single path-only-meta bullet correction
- Existing tests must still pass
</constraints>

<verification>
```bash
grep -n 'self.meta\[str(new_idx)\]' src/semantic_search/indexer.py   # path-only write in add_file_to_index
grep -n 'Full content in meta' CLAUDE.md                             # must return nothing
grep -n 'Path-only meta' CLAUDE.md                                   # must return the corrected bullet
make precommit                                                       # must exit 0
```
</verification>
