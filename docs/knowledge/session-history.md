# Session History

Reverse-chronological: newest entry at the top, one bullet per session, appended at session wrap (`dev-session-wrap` step 4). Each entry is a dense audit paragraph written for a cold reader. Headline: `- **YYYY-MM-DD (session size, agent — what shipped in one clause)** —`. Body covers, in rough order: what landed with **commit hashes** (hashes live here, not in `PROGRESS.md`), design corrections the user made mid-session (quote the load-bearing ones verbatim), the self-reflection — what failed and what worked, concretely, citing the specific moment rather than "went well overall" — any new memories/learnings/invariants recorded, and the next-session entry point. Don't sanitize failures: the entry's value is the honest record the next session orients from.

Entry template:

```markdown
- **2026-01-15 (long, Claude — feature X shipped end-to-end; design reshaped mid-session)** —
  Built X per the approved plan (`abc1234` feat, `def5678` docs); user redirected the
  placement design after the first proposal ("ground it in the real surface"), which became
  a learnings entry. Verification: 42/42 tests, browser pass confirmed the two user-visible
  flows; one regression found and fixed in-session (`9abcdef`). Self-reflection: anchored on
  the prompt's framing for the first scope draft — caught at the spec review gate, not before;
  the per-scenario verification split worked, surfacing the regression scenario 2 would have
  masked. New learning: <headline>. Next session: phase 2 (UI), plan is `Ready` in INDEX.
```

---

_No sessions recorded yet._
