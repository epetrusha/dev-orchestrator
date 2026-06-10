---
name: dev-build
description: "Use after dev-writing-plan has produced an approved plan. Execution-phase discipline: load TodoWrite from plan, dispatch per the plan's briefs, capture evidence, transition to the Prove pass. Triggers: 'execute the plan', 'implement', 'start building', 'do the work'. Do NOT load for: spec writing, plan writing, investigation, verification, session-wrap."
---

# Dev Build

Execute an approved plan with TodoWrite-tracked tasks, subagent discipline per the plan's authored briefs, and evidence capture.

## Pre-conditions

- Approved plan in `docs/planning/plans/active/` with `Status: Ready`, explicitly approved by the user

If pre-conditions fail → STOP. Return to writing-plan or wait for approval.

<HARD-GATE>
Any change to plan, scope, file map, or verification matrix → STOP, surface to user, wait for explicit assent. Updating the plan file alone is insufficient.
</HARD-GATE>

## Checklist (TodoWrite-tracked, in order)

1. **Re-read the plan.** Quote the goal + verification matrix row count into the TodoWrite closure.

2. **Load TodoWrite from plan.** EVERY plan item (code tasks, verification matrix rows, docs updates, commit/push) becomes a todo. Items not in TodoWrite get dropped silently.

3. **Declare orchestration per task.** `inline | subagent (which subagent, model tier)`.

4. **Dispatch subagents per plan's authored brief.** Briefs were authored in writing-plan (Task body). Don't re-author. Verify the brief includes exact paths for any spec/plan/code the subagent must read.

5. **Execute one todo at a time.** Mark `in_progress` before starting, `completed` only when actually finished.

6. **After subagent work:** read the diff, not the report. Grep expected symbols/events/consumers across packages.

7. **Capture verification matrix evidence inline.** Each row's done-check evidence (script output, screenshot path, test count) goes into the TodoWrite closure for that row.

8. **On test failure or any plan deviation:** STOP. Re-read project CLAUDE §Degradation prevention + `.claude/skills/dev-orchestrator/references/cognitive-discipline.md`. Surface to user before iterating.

9. **Transition to the Prove pass** (verification) — see the orchestrator's §Prove.

Per-10-tool-call self-check is self-discipline: every ~10 calls, state in one line what you're doing and whether it matches the plan.

## Cross-cutting rules

Communication discipline, engagement-vs-approval, hard-correction handling, length ceilings, jargon discipline — defined in `dev-orchestrator`. Re-read that block before every user-facing message.
