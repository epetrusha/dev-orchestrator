---
name: dev-session-wrap
description: "Use at end of session — enforces the 5-step session-end checklist (docs, commit, self-reflect, notes, memory/skill updates). Triggers: '/dev-session-wrap', 'wrap up', 'end of session', 'let's stop here'. Do NOT load for: mid-session work."
---

# Dev Session Wrap

5-step session-end discipline with DoD per step. Replaces honor-based wrap-up.

## Checklist (TodoWrite-tracked, in order)

1. **Update docs.**
   - Append session entry to `docs/PROGRESS.md` (date + one-paragraph summary). Do NOT embed commit hashes — git log already carries them, and embedding forces placeholder-and-amend cycles for zero value. The knowledge base's `session-history.md` keeps hashes by separate convention (step 4).
   - Move shipped items from `docs/BACKLOG.md` to `docs/FEATURES.md`
   - Add any new bugs/issues surfaced this session to `docs/BACKLOG.md`
   - Update `docs/planning/INDEX.md` if any plan/spec files moved between active/shipped/abandoned
   - Update `docs/INVARIANTS.md` and the project's domain docs (per the `CLAUDE.md` Documentation map) if behavior or invariants changed
   - Walk the active/shipping plan file: tick `- [ ]` → `- [x]` for each sub-step whose code shipped this session. If any sub-step body still describes an approach that didn't ship (signature, file path, code block, dropped task without inline note) — update it inline. The plan body must read true at the moment of `git mv` to `plans/shipped/`; the orchestrator's Build HARD-GATE requires per-commit propagation of approved deviations, but checkbox-ticking and any missed propagation get a final sweep here.

   **DoD:** each affected doc has a new entry or removed line visible in `git diff`. The shipping plan has zero un-ticked sub-step boxes for code that landed this session AND its body describes what actually shipped. Yes/No: docs reflect today's work AND the shipping plan body matches the shipped code.

2. **Commit on dev.** Stage exact files (no `git add -A`). Commit message follows project style. Push.

   **DoD:** `git log -1` shows today's commit with proper message; `git push` succeeded (`git status` shows clean tree, no unpushed commits).

3. **Self-reflect.** Write a concrete paragraph: what went well, what went poorly, what to do differently. Generic phrases ("things went OK", "no major issues") are NOT acceptable.

   **DoD:** the reflection cites a specific moment, decision, or failure mode with enough detail that a future session could act on it.

4. **Update the knowledge base at the location declared in project `CLAUDE.md` §Knowledge base** (in-project `docs/knowledge/` by default; may be an external vault/wiki). Format primer: `docs/knowledge/README.md`. No location declared → close as explicit `N/A: no knowledge base`.
   - Session history — **always** append an entry per its header format (date + commit hashes + honest self-reflection + next-session entry point)
   - Learnings — **only if** self-reflection surfaced a generalizable lesson per the litmus tests at the top of that file (would the principle help in a different domain? non-software example? delete session-specific nouns and does it stand?). Decide actively; "no new learnings" is a valid outcome.
   - If the knowledge base is an external Obsidian-style vault: also refresh its `_index.md`; never edit `planning.md` (human-owned).

   **DoD:** the knowledge base shows the new entry (new commit if versioned, or explicit log entry that nothing changed today) — or the explicit `N/A`.

5. **Update memory / CLAUDE.md / skill files if a persistent rule emerged.** Per the "Hard-to-find-in-docs is a docs failure" learning, gaps surfaced this session get patched THIS session, not deferred.

   **DoD:** `git log -1` shows the persistence update OR explicit decision "no new persistent rule this session."

## Output

Single-content message: "Session wrap complete. Commits: <hashes>. Notes updated. <learnings count or 'no new learnings'>. State cleared."

## Cross-cutting rules

Communication discipline, engagement-vs-approval, hard-correction handling, length ceilings, jargon discipline — defined in `dev-orchestrator`. Re-read that block before every user-facing message.
