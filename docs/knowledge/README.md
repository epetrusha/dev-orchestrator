# Knowledge base

Durable cross-session memory about *how the work goes* — distinct from `docs/` (the state of the project) and `docs/planning/` (the state of the work). `dev-session-wrap` step 4 writes here.

**Where it lives is your call — declare it in `CLAUDE.md` §Knowledge base.** Three options: in-project at `docs/knowledge/` (this default — versioned with the code, zero extra tooling), an external vault/wiki (e.g. an Obsidian folder per engagement — keeps reflections out of a public repo and aggregates across projects), or none (the wrap step closes as explicit N/A). The format below is the contract either way.

## Files

- [`session-history.md`](session-history.md) — reverse-chronological session audit trail. **Commit hashes live here**, not in `docs/PROGRESS.md` (embedding hashes there forces placeholder-and-amend cycles; here the entry is written once, after the commits exist).
- [`learnings.md`](learnings.md) — generalized, transferable principles distilled from self-reflection. The highest-value file in the layer and the easiest to ruin — see the priming at its top before writing.

## If stored in an external Obsidian-style vault

Two extra conventions apply there (not needed in-project):

- `_index.md` — the engagement's landing note: frontmatter (`tags`, `category`, `status`, `description`, `path:` pointing at the code repo) + a one-paragraph summary + wikilinks to the sibling files.
- `planning.md` — **human-owned**. The user's own strategy/idea notes. AIs read it for context and never edit it.

## Writing discipline

Both files are append-mostly and written for a cold reader six months out. Session-history entries are honest audit records — failures and corrections included verbatim where load-bearing, because sanitized entries teach nothing. Learnings entries are principles with every session-specific noun deleted; if an entry only helps when the exact same situation recurs, it isn't a learning, it's a diary line in the wrong file.
