---
name: dev-investigate
description: "Use when user request needs code-reading to know what's wrong / where the boundary is. Diagnose-first read-only mode. Triggers: vague reports ('looks weird'), 'why does X happen', 'trace this', 'find what's causing', 'investigate before fixing'. Do NOT load for: explicit task requests with clear scope, already-planned work."
---

# Dev Investigate

Read-only diagnostic phase. Produces findings the user can decide next-phase from.

<HARD-GATE>
No code edits, no plan-writing, no subagent dispatch while in investigate phase. Read-only.
</HARD-GATE>

## Checklist (TodoWrite-tracked, in order)

1. **Restate symptom in one sentence.** Include user's exact words verbatim.

2. **Search code for relevant surfaces.** For each find: TodoWrite item closed with quoted lines + `file:line`. If the project keeps symbol manifests or interface indexes, read those first (cheap pointers), then the code.

3. **Form a hypothesis tree.** Top 2-3 hypotheses for what could cause the symptom. Each with evidence-or-falsification criteria.

4. **Narrow via inspection.** Read code at each hypothesis branch; kill or confirm.

5. **State conclusion.** What the actual cause is, with `file:line` evidence inline.

6. **Decide next phase with user.** Surface 2-3 options:
   - **Inline fix** if 1-3 files and intent is now explicit → user can approve and orchestrator writes inline mini-plan
   - **Brainstorm + plan** if design decisions involved → invoke `dev-brainstorm`
   - **File to BACKLOG** if pre-existing and not in this scope (explicit user assent required)

7. **Wait for user pick.** Engagement is not approval; explicit choice required.

## Output artifact

- **Simple investigations** (single hypothesis, quick read): findings inline in chat.
- **Complex investigations** (multi-file, multi-hypothesis, design implications): write to `docs/planning/investigations/YYYY-MM-DD-<symptom>.md`. This file is read by `dev-brainstorm`'s Read-project-context step as a doc to incorporate.

Frontmatter for investigation files:

```yaml
---
Date: YYYY-MM-DD
Symptom: <user's words>
Status: Open | Resolved
---
```

Sections: Symptom, Hypotheses considered, Conclusion + evidence, Next-phase options.

## Cross-cutting rules

Communication discipline, engagement-vs-approval, hard-correction handling, length ceilings, jargon discipline — defined in `dev-orchestrator`. Re-read that block before every user-facing message.
