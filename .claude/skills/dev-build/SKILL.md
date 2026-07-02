---
name: dev-build
description: "Use after dev-writing-plan has produced an approved plan. Execution-phase discipline: load TodoWrite from plan, dispatch per the plan's briefs, capture evidence, transition to the Prove pass. Triggers: 'execute the plan', 'implement', 'start building', 'do the work'. Do NOT load for: spec writing, plan writing, investigation, verification, session-wrap."
---

# Dev Build

Execute an approved plan with TodoWrite-tracked tasks, subagent discipline per the plan's authored briefs, and evidence capture.

<ENTRY-GUARD>
Run only if `▸ pipeline-position: build (via orchestrator)` was printed above this invocation in the current session (rerouting back from Prove counts if this skill was the originator). Absent → you were reached outside the orchestrator; stop and hand back to it. You are never an entry point — the orchestrator is the only door.
</ENTRY-GUARD>

## Pre-conditions

- Approved plan in `docs/planning/plans/active/` with `Status: Ready`, explicitly approved by the user

If pre-conditions fail → STOP. Return to writing-plan or wait for approval.

<HARD-GATE>
Any change to plan, scope, file map, or verification matrix → STOP, surface to user, wait for explicit assent. Updating the plan file alone is insufficient.
</HARD-GATE>

## Checklist (TodoWrite-tracked, in order)

1. **Re-read the plan.** Quote the goal + verification matrix row count into the TodoWrite closure.

2. **Load TodoWrite from plan.** EVERY plan item (code tasks, verification matrix rows, docs updates, commit/push) becomes a todo. Items not in TodoWrite get dropped silently.

3. **Load the plan's orchestration.** Take the per-task `inline | subagent + model tier` from the plan's file map; don't re-derive it. A task the plan left without a tier is a plan gap → surface it before dispatching.

4. **Dispatch subagents per plan's authored brief.** Briefs were authored in writing-plan (Task body). Don't re-author. Verify the brief includes exact paths for any spec/plan/code the subagent must read.

5. **Execute one todo at a time.** Mark `in_progress` before starting, `completed` only when actually finished.

6. **After subagent work:** read the diff, not the report. Grep expected symbols/events/consumers across the codebase.

7. **Capture verification matrix evidence inline.** Each row's done-check evidence (script output, screenshot path, test count) goes into the TodoWrite closure for that row.

8. **Execute the plan's intent; if off-rails, stop and run the drill.** Build the plan as written; don't relitigate settled design mid-build. When its shape won't implement, serve its intent another way — never defer the feature or narrow its scope to resolve "can't build it as written." Run the off-rails drill (`dev-orchestrator §Off-rails`); its surfacing is the only sanctioned deviation. Re-read project CLAUDE §Degradation prevention + `.claude/skills/dev-orchestrator/references/cognitive-discipline.md` when it triggers.

9. **Transition to the Prove pass via the orchestrator** (§Prove; the project's declared harness skill if `docs/TESTING.md` names one). Prove is a pass you ENTER — never run tests/audits/UI checks ad hoc in its place.

9b. **On the Prove report.** Clean → done; let the pipeline continue. An issue → triage by altitude: **mechanical** (one correct answer — typo, wrong flag, missing import) → fix it; **design-level** (more than one defensible resolution, turns on architecture/principle) → run the drill (`dev-orchestrator §Off-rails`), which returns here to implement the resolved fix. Either way, **resume Prove via the orchestrator afterwards** — re-enter the pass to verify the fix and, when it returned early on a blocker, to run the verification that blocker prevented. Prove is done only when the full verification matrix passes, not when the blocker is merely cleared.

Per-10-tool-call self-check is self-discipline: every ~10 calls, state in one line what you're doing and whether it matches the plan.

## Cross-cutting rules

Communication discipline, engagement-vs-approval, hard-correction handling, length ceilings, jargon discipline — defined in `dev-orchestrator`. Re-read that block before every user-facing message.
