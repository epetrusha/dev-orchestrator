---
name: dev-external-agent
description: "Use when implementing a planned change via an EXTERNAL coding agent instead of an internal subagent — driving an external executor (resolved by tier from the executor registry), or authoring the prompt for one. Triggers: 'write a codex prompt', 'dispatch this to an external agent', 'implement this plan via an external agent', 'finish phase X via an external agent'. The orchestrator authors the grounded prompt and reviews the diff; the executor implements. Do NOT load for: internal-subagent implementation (use dev-build), spec/plan writing, verification."
---

# Dev External-Agent

Implement a planned change by driving an external coding agent instead of an internal subagent. You author the grounded prompt; the executor produces a diff; you review the **diff** — never the report — gate on your own test pass, and integrate.

<ENTRY-GUARD>
Run only if `▸ pipeline-position: build-external (via orchestrator)` was printed above this invocation in the current session (rerouting back from Prove counts if this skill was the originator). Absent → you were reached outside the orchestrator; stop and hand back to it. You are never an entry point — the orchestrator is the only door.
</ENTRY-GUARD>

<HARD-GATE>
The executor's changes reach the integration branch ONLY through your review. It works in an ISOLATED locus — an unstaged working tree, or its own run branch/worktree — and never pushes to, merges into, or rewrites the integration branch. You read the diff (not the agent's self-report), run the suite yourself, and you alone integrate reviewed-green work. Agent-green ≠ integrated. Holds for every transport, executor, and isolation model.
</HARD-GATE>

## Three pluggable axes — fix none as invariant

The doctrine below holds across all of these; only the dispatch + integrate mechanics change.

- **Transport** — how the prompt reaches the executor: **headless** (the orchestrator runs the executor's CLI and reads the diff + output — the command is resolved by tier from the executor registry, never named here) or **courier** (a human pastes prompt → IDE chat → output back). Courier is one option, not the spine.
- **Executor** — what runs the prompt: pick a **tier** from the executor registry (`dev-orchestrator/references/executors.md`) by whether it brings its own harness: **`agent`** (harness+model — edits files, returns a diff you review; the doctrine below applies in full; earns its keep only as a *different* model than the orchestrator), **`model`** (a bare model — returns text, no edits; *you* are the harness: apply + own the output under your normal edit discipline, there is no diff to review), or **`self`** (you do it inline). Match the tier to the task's risk.
- **Isolation** — where the executor's changes land, and how you integrate them:
  - **Shared working tree** — same branch, same machine; the executor edits in place and leaves changes UNSTAGED. Simplest; fine when you drive it synchronously and own the tree. Cost: it can disturb your live tree, and concurrent runs collide.
  - **Dedicated branch / worktree** — same machine, isolated tree; the executor may commit on its own run branch. Use for autonomous, concurrent, or cleanly-revertible runs; you review the branch diff and merge.
  - **Remote** — cloud agent or CI runner, a different machine; produces a patch or PR. Use when the executor isn't on your machine; you review the PR diff, run tests on the branch locally, then merge.

## Sequence (TodoWrite-tracked, in order)

1. **Scope the run.** One coherent change; ceiling ≈ one phase. Size to the executor's context window, not a 1M assumption — split if larger; the executor degrades silently near its ceiling. Never a lone pure-core task with nothing to integration-test — pair it with the boundary task that gives it a reviewable gate. Hold the gnarliest task (state-machine pause/resume, security/permission boundary, migration-bearing) for its own run. First run of a new executor or transport: calibrate tight for a clean diff, then scale on live feedback.

2. **Ground the whole flow yourself — the orchestrator does not re-ground before offloading here.** The plan's citations are upstream and secondary; the executor inherits only what your prompt carries, so live grounding happens at this step or nowhere. Meet the full bar of orchestrator `§Grounding`: trace end-to-end what the prompt will direct — every layer, seam, consumer, processor, and input variant — each a primary-source fact (code / config / canon) confirmed this session, not a `file:line` relayed from the plan or remembered. Step 3's allowlist is this map made concrete: an untraced downstream file won't get allowlisted.

3. **Author the prompt** — self-contained and editable. Apply every convention:
   - **Allowlist the WHOLE change.** Exact files PLUS the downstream a change necessarily drags in: the index/barrel re-export when a new shared symbol is added; the completeness/registration test that mirrors a declared registry when a new entry registers; the locale/resource bundle for a new user-facing key. Every file whose `file:line` the prompt cites is an edit target — list it. Forbid unrelated refactors: the rule is "everything THIS change needs," not "everything."
   - **Name a release valve.** If the change genuinely needs an edit outside the allowlist, the executor makes it AND flags it under a `VALVE-RELEASE` heading (file + why). A half-change that stops at the boundary — or leaves a gate red — is worse than a flagged extra edit. Valve entries are the first items on your review checklist: scope and design drift hide there.
   - **Resolve foreseeable forks.** Where the work forks between an expedient local patch and the system-consistent shape, name the fork and the intended resolution in one line — the executor lacks the architectural frame and defaults to expedient, eroding an invariant the codebase just established. Litmus: if the tests pass two ways and one undoes a principle the work is enforcing, the prompt must say which way.
   - **Quote the plan; don't re-sketch it.** A paraphrase compresses a constraint into a bug. State exclusions ("X owns this — exclude it"), not just surfaces, or the executor re-handles and regresses what is already handled. Never put a literal object shape in a prompt unless it is correct — describe it semantically.
   - **Permissions + return contract.** Git: the executor writes only to its isolated locus and never pushes to, merges into, or rewrites the integration branch — in a shared tree it leaves changes unstaged; on its own run branch/worktree it may commit there. Let it run tests so it catches its own regressions — you still re-run as the gate. It returns per-file changes + assumptions + blockers, and claims no verification beyond a test it actually ran.

4. **Dispatch.** Set up the isolation locus first if it isn't the shared tree (cut the run branch/worktree; or, remote, point the run at its branch). Then: headless — run the chosen tier's command (from the executor registry) and capture the diff + output; courier — hand the prompt to the human → executor → output back.

5. **Review the diff, not the report.** Grep for the expected new symbols/entries/registrations; trace the integration across the project's data-flow layers (project `CLAUDE.md` §Data-flow discipline). Run the cheapest layer that proves the contract the user traverses — unit tests are necessary-not-sufficient when an integration boundary consumes the shape under change. **Divergence here is off-rails, not a patch-and-integrate** — triage it per §Triage & attribution below. When the shape won't build, serve the plan's intent another way (re-prompt, re-scope the run), never defer the feature or narrow its scope.

6. **Gate, integrate, attribute.** Run the tests yourself at the right layer (`docs/TESTING.md`), then run the live real-usage sweep through the orchestrator's Prove pass (§Prove; the project's declared harness if one exists — routed via the orchestrator, which sets the position marker before entry). A run is green only after your own pass and the Prove pass, never the executor's claim. Each surfaced issue runs the two-pass §Triage & attribution below — resolve it first (design-level → off-rails/deliberator), then attribute its prevention link. Integrate reviewed-green work onto the integration branch (commit in a shared tree; merge a run branch / PR).

## Triage & attribution — resolve the issue, then route its prevention

Each issue this run surfaces — at the gate (diff-review), or when a Prove report routes back to you as the originator (prove re-entry) — gets TWO passes, in order; never collapse them:

**Pass 1 — resolve (off-rails first).** Triage by altitude before touching anything:
- **Design-level** — a seam that's a hack/fallback/special-case, a plan/canon fact the run proved false, a fork between an expedient patch and the system-consistent shape, a blocker that the shape can't be built as written. → **STOP and run orchestrator `§Off-rails`**: reground, then hand the deliberator **intent + gap + grounded facts ONLY**, and freeze. Do NOT pre-define the forks, seed candidate shapes, or fold it into the integration commit — and do NOT hand down your perceived limitations (what "can't be done", is "out of scope", or "too costly") as facts: those are hypotheses for the deliberator to ground, not constraints that pre-poison its search. A NEW grounded fact found while implementing the resolution is itself a fresh off-rails — hand it back. Surface the deliberator's resolved design (translated to the user's altitude) and WAIT for explicit approval before the first edit.
- **Mechanical** — a typo, wrong flag, missing import, a count/index/locale entry the change mechanically drags in, a transient flake. → fix inline and continue (no drill).

The litmus: is there more than one defensible resolution, and does choosing turn on architecture/principle rather than a single correct mechanical answer? If yes — you'd be *picking*, not *correcting* — it is design-level: drill it.

**Pass 2 — attribute (only after the issue is resolved).** Name where the resolved issue should FIRST have been caught — to route the *prevention*, not to blame, and not as the fix itself. Chain **Plan → Prompt → Execution** (mirrors orchestrator `§Off-rails`):
- **PLAN-GAP** — the plan was wrong/ambiguous/stale here → amend the plan.
- **PROMPT-GAP** — the plan was adequate; the prompt lost, garbled, or under-emphasized a constraint, or sketched a buggy structure in an example → tighten a step-3 convention.
- **EXECUTION-GAP** — plan + prompt adequate; the executor misimplemented anyway (regression, misread, shortcut, scope drift) → tighter prohibition, a self-check it runs, or model choice.

Assign one PRIMARY category (earliest link) + note contributing links. A prompt-gap with an execution assist is still primary-prompt — the prompt set the trap.

After resolving and attributing, **resume Prove via the orchestrator** — re-enter the pass to verify the fix and complete any verification a blocker cut short; Prove is done only when the full matrix passes.

## Run records

Keep a prompt + diff + review verdict per run. Where they live is transport-dependent — courier runs file them under `docs/planning/orchestration/runs/run-NN/`; headless runs may capture inline. Per-run learnings and observed executor context-window sizes accrue in `docs/planning/orchestration/external-agent-playbook.md` (history; this skill is the doctrine).

## Cross-cutting rules

Communication discipline, engagement-vs-approval, hard-correction handling, length ceilings — defined in `dev-orchestrator`. Re-read that block before every user-facing message. Grounding and off-rails are execution disciplines homed in the sequence above (steps 2 and 5), not comms rules — don't re-file them into this block.
