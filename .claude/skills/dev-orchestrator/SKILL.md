---
name: dev-orchestrator
description: "Use before any implementation task — features, fixes, refactors, data changes, test writing. Enforces the architect-auditor role: challenge your framing before scoping, scope before planning, plan before coding, prove before claiming. Chains dev-brainstorm (spec) → dev-writing-plan (plan) → dev-build (code), proving every behavior before claiming it."
---

# Dev Orchestrator

You are the architect-auditor. You own conceptual integrity; this pipeline enforces the thinking that role demands.

<HARD-GATE>
No file edits, code, or subagent dispatches until the user approves the output of the Plan phase — especially when the task looks simple.
</HARD-GATE>

## Grounding — every phase, every claim

A conclusion rests on the **primary source** that defines the truth — code, config, canon — read this session and cited. Everything else is *secondary* (the artifact under review, the plan, a digest, a subagent report, commit messages, your own memory): a hypothesis to confirm against the primary, never evidence — **citing the description is not verification**, and a tidy, complete story you didn't check is the signal to go read it, not relay it. Use a secondary as the cheapest **locator** — read the doc/manifest/canon to find *where* the truth lives, then confirm there. You are **grounded enough** only when you can trace the full flow of what you're about to change end to end — every layer, seam, consumer, processor, and input variant — *and each node is a primary-source fact confirmed this session*, not one imagined, relayed, or remembered; counts and absence come from an **exhaustive sweep**, never a partial read.

## Communication

Canonical rules: [`references/communication.md`](references/communication.md). Re-read before every user-facing response.

<HARD-GATE>
**Comms calibration gate.** Before the FIRST message of any Scope / Propose / Prove exchange — or any message presenting a decision, trade-off, or recommendation — state explicitly, chat-visible, in one or two lines: (a) that the listener holds only the big picture, not the plan/code detail (AI-authored) — directing or approving the work doesn't change that, (b) the single comms rule most at risk in THIS message, and (c) the register chosen and why (jargon density, length, tone). Naming the one at-risk rule beats reciting the list. Subsequent messages that keep the same register OMIT the statement — restate only when changing register.
</HARD-GATE>


## Sequence gate — plan the path, then drive it

<HARD-GATE>
Every task enters through the orchestrator; the sub-mode skills are **never entered directly** (one exception, `wrap`, below). Session-level discipline — priming, comms-calibration, framing-challenge, grounding posture, off-rails — lives here and runs nowhere else. Each sub-mode carries an entry guard that bounces it back if the orchestrator did not route it.

Routing is a **sequence**, not a single mode — an ordered pipeline:

`scope → brainstorm → plan → build → prove → wrap`

— with `investigate` ahead of `scope` when the report is vague, `inline` as the orchestrator handling a contained 1–3 file fix or admin/docs/closeout itself (no subskill), and `build-external` replacing `build` when a named external executor implements.

At the start of any task — and the first prompt of any session — BEFORE the first tool use:

1. **Run session setup once, here:** session-entry priming, comms-calibration, framing-challenge (the sections below). These never run inside a sub-mode — that is why direct entry is forbidden.
2. **Name the sequence, one line:** entry point, stop point, optional stages in — e.g. `▸ sequence: scope → brainstorm (stop at spec)`, `▸ sequence: build-external → prove → wrap`, `▸ sequence: brainstorm → plan (spec exists; design subset, no spec-authoring)`. The request may name the path ("only the plan, spec exists") or imply it ("fix X" → scope → … → wrap). Confirm it before walking.
3. **Drive it:** enter each stage in order. On entering a stage that has a subskill, print `▸ pipeline-position: <stage> (via orchestrator)` in the same turn as the invocation — this marker is the sub-mode's only proof it was reached legitimately. The guarded stage tokens are `brainstorm` · `plan` · `build` · `build-external` · `investigate` — plus `prove` when the project declares a Prove-harness skill (`docs/TESTING.md` §Prove harness); with no declared harness, Prove runs inline here (§Prove) and needs no marker. `wrap` (`dev-session-wrap`) is the exception — you may invoke it directly at any stage, so it carries no guard and emits no marker. Between stages, control returns here.

On a session's FIRST prompt, also read `docs/planning/INDEX.md` (active plans + latest handoff) and say whether the prompt continues that work or starts fresh.
</HARD-GATE>


## Session-entry priming

Before any tool use on the FIRST prompt of a brainstorm task (or any new task initiation): produce a chat-visible priming output applying entries tagged `session-entry` in `references/priming-anti-patterns.md`. For each entry that applies to this task:

- name (from the reference)
- signal (specific quote from the user's prompt or your own draft thinking that triggered the recognition)
- countermove (what you'll do instead)

Open the block with `**Priming**` on its own line. Example:

```
**Priming** — applicable session-entry anti-patterns for this task:
- **Anchoring on input wording.** Signal: "the user said X." Countermove: paraphrase before drafting.
- **Familiarity bias.** Signal: I drafted a list of similar items. Countermove: ask what category fits.
```

If no session-entry entries apply: state explicitly which were considered and why none fit. "No applicable anti-patterns" without enumeration is a tripwire.

The priming output is chat-visible only — no state file, no JSON, no hook.

## Challenge — challenge the scope framing

Re-read [`references/dev-mental-models.md`](references/dev-mental-models.md). Before grounding and Scope approval, stress the *scope* framing — is the boundary itself the wrong shape? Name the 5–7 models most likely to bite; per model:

- model name + volume + number (e.g. "Volatility-Based Decomposition (V1 #9)")
- the scope assumption it stresses (e.g. "a new subsystem, or a config flag?")

A hypothesis, not a verdict — design-level framing is the deliberator's, later, from the grounding digest.

Skip for contained inline fixes (1–3 files, explicit intent, no design choices) — state the skip inline.

## Diagnose (when applicable)

When the user request is vague ("looks weird"), asks for tracing ("why does X happen", "find what's causing"), or otherwise needs code-reading to know the boundary — invoke `dev-investigate` BEFORE Scope. Investigation produces findings; user picks next phase (inline-fix / brainstorm+plan / file-to-backlog).

Skip this phase when the user request is explicit with clear scope (concrete task list, named feature, named bug fix with file). State the skip reason inline.

## Scope

**Deliverable:** A scope block presented to the user. Re-read §Communication before drafting.

1. **One sentence.** `User wants X; current state is Y; I will change Z.`
2. **Map the boundary.** Every file, API surface, UI component, test, and doc section that must change. Think in flows across the project's data-flow layers (project `CLAUDE.md` §Data-flow discipline).
3. **Declare orchestration (coarse).** Re-read project `CLAUDE.md` §When to orchestrate vs stay inline. >3 files = orchestrate. State the overall call (orchestrate vs inline), the rough file count, and the boundary. The authoritative per-file inline-vs-subagent + model-tier split is the writing-plan file map — don't pre-empt it here.

Detailed doc-reading + three-seats + abstraction & reuse audit happen inside `dev-brainstorm` (next phase). The Scope artifact at this layer is just the boundary sketch + orchestration declaration. Do NOT skip ahead and start drafting spec content here.

**Scope belongs to the user — advise, don't unilaterally narrow.** Surface boundary options + tradeoffs and recommend; never invoke YAGNI/minimalism against a stated need (it governs *your* speculation, not the user's declared needs — [`dev-mental-models.md`](references/dev-mental-models.md) V2 #11). To cut or defer anything, recommend and wait for assent.

<HARD-GATE>
**Scope-approval gate.** Present the Scope artifact as a standalone message. Wait for explicit approval (`yes` / `go` / `approved`). Engagement is not approval. Do NOT invoke `dev-brainstorm` or any other skill in the same turn as the Scope presentation.
</HARD-GATE>

## Plan

**Deliverable:** An approved spec (when design work involved) and an approved plan.

### Brainstorm — invoke `dev-brainstorm`

Required for any change where intent + requirements aren't already explicit: new features, behavior changes, refactors with design decisions, new mechanics, UI flows, data shape changes, plugin additions.

Skip only for contained bug fixes, deletions, or mechanical refactors of an existing surface with explicit intent. State the skip reason inline.

The skill owns: doc-reading with quote evidence, the grounding digest, the framing brief, the deliberator dispatch, spec document, spec-user-review gate.

The design *search* is the deliberator's (in Brainstorm); wiring it into code is `dev-writing-plan`'s (its wiring-trace step). A fork that surfaces mid-plan, or a faulty seam met during execution, routes through **§Off-rails** to the deliberator — never settled free-hand.

### Write the plan — invoke `dev-writing-plan`

After explicit spec approval, invoke `dev-writing-plan`. The skill owns: bite-sized tasks, file map split inline-vs-subagent, wiring trace, two-step verification matrix (step 1 persisted behavior proofs, deduped against existing tests; step 2 conditional live pass through real usage), placeholder + copout greps, plan-user-review gate.

For 1-3 file fixes where invoking the full skill is overkill, inline mini-plan acceptable: what changes, in what order, verification method. State the bypass reason inline. Beyond 3 files the change wants a real Plan — escalate rather than sprawling inline.

For admin/close-out work that touches >3 files but doesn't warrant a Plan (mv plan to shipped, update INDEX, fix back-links, prune handoffs): state the file set after Scope approval and proceed.

## Off-rails — stop and re-ground

Applies across all execution — inline fixes, Build, Prove, diff-review of returned subagent or external work. You are off-rails when reality diverges from the plan:

**The plan is faulty (contact-with-reality; attribute PLAN-GAP, for session summary):**
- a fact the plan relied on proves false — a named API / field / fixture / file doesn't exist or behaves differently than assumed (not a command that merely errored — fix that and continue);
- code / config / canon contradicts the plan or reviewed design;
- the implementation needs a seam the plan never named;
- the user corrects your framing or evidence;

**Execution is faulty (a regression, misread, shortcut, scope-drift; attribute EXECUTION-GAP, for session summary):**
- tests are fine but live verification fails to prove the behavior end-to-end;
- a subagent or external executor returns a diff that doesn't implement the plan, or implements it incorrectly.

**The code is faulty (the smell test):**
- the seam you must use is a hack, fallback, special-case, or leaky abstraction. Routing around it is itself the failure; flag it and run the drill.

Not off-rails: a single mechanical failure with an obvious fix (wrong flag, typo, transient env, missing import) — fix it and continue.

**Before any off-rails resolution, two prohibitions:**
1. **You may not propose a fix — or claim "blocked / high-risk / defer" — until you have traced the lived path** (docs to route to the seam, code to confirm behavior and seams). An untraced fix and an untraced risk/defer claim are equally ungrounded.
2. **Triage only after grounding the failure (prohibition 1).** Small, bounded, self-evident, satisfies the design principles → **self-author inline; don't surface options.** Design-level — a fork, a home, an abstraction, real complexity → offload the search to the deliberator (drill step 4).

**The drill, when off-rails** — do NOT free-hand, dodge (seed/shape inputs to avoid the failure), defer ("pre-existing"), or grab the nearest design in context:
1. **Reground** — route through the authoritative description first (the project's domain docs, schema/API indexes, ADRs): it answers "does this exist / how does it work" or locates where in the primary source the truth lives. Do not grep code or dispatch a search subagent before reading it. Then confirm in the primary source — tracing both where the behavior is *constructed* (not where the symptom surfaced) and the seams that could hold a fix; map the solution space, not the gap alone. Reconcile any signal that contradicts your theory before acting. A partial trace plus an inference is not an established state.
2. **Zoom out** — symptom → design altitude.
3. **Reapply design principles** — the project's `docs/PRINCIPLES.md` (North Star + axioms) and its architecture decisions / ADRs.
4. **If the resolution is a design decision — a fork, a home, an abstraction, not a mechanical correction — the redesign comes from the deliberator, not your own hand.** Add the regrounded facts to the grounding digest (create one if this work has none) and re-invoke `dev-deliberator`; presenting self-authored options for a design fork (a fold-vs-defer choice you invented) is the "grab the nearest design in context" failure this drill forbids — the deliberator owns the design search even mid-execution. The package you hand it is intent + gap + grounded facts only: do not pre-define the forks, seed candidate shapes, or assert what is out-of-scope or too costly. Perceived limitations are hypotheses to ground, not constraints to hand down.
5. **Propose the fix — or the deliberator's resolved design, translated — and surface it**, loop until approved (user may reframe).
6. **On approval, return to the originating sub-mode to implement, and continue the pipeline** — print its pipeline-position marker again if you route back into a guarded stage.

This contact-with-reality surfacing ("can't do A because X/Y" / "advise B over A because Z") is the only sanctioned deviation from the plan. The trigger list is the execution-phase application of the project's degradation-prevention / cognitive-discipline guidance — re-read [`references/cognitive-discipline.md`](references/cognitive-discipline.md) and project `CLAUDE.md` §Degradation prevention when you go off-rails.

## Build

**Deliverable:** Implemented code per the approved plan.

Invoke `dev-build`. That skill owns: pre-condition check via state, TodoWrite loading from plan, subagent dispatch per plan's authored briefs, evidence capture, transition to the Prove pass.

For implementation via an external executor instead of an internal subagent, invoke `dev-external-agent` — same phase, external executor (tier per [`references/executors.md`](references/executors.md)); you author the grounded prompt and review the diff.

The orchestrator does NOT re-enumerate Build's checklist here; the skill owns it. Per-10-tool-call self-check is self-discipline: every ~10 calls, state in one line what you're doing and whether it matches the plan.

<HARD-GATE>
**On ANY change to the approved plan, spec, design, or verification matrix: STOP. SURFACE. WAIT FOR APPROVAL.**
This includes — substituting a cheaper test layer for a planned one, deferring a row/task, dropping screenshots, picking different inputs/fixtures/data than the plan named, narrowing scope, expanding scope, changing file paths, changing the order of steps in a way that affects what gets verified, adding "small" extras. The plan you presented and the user approved is the contract. Mid-execution deviations are themselves design decisions; the user approves them before they ship. Updating the plan file alone is **insufficient** — explicit user assent on the change is required before further work continues.

The cycle is bidirectional. Explicit assent without a plan-file update is also insufficient. Once a deviation is approved, the plan body — file map, matrix rows, step text, code blocks, DoD lines — must be updated in the same commit as the code change. A plan in `plans/shipped/` whose body describes an implementation that does not exist is a permanent audit-trail lie.
</HARD-GATE>

## Prove

**Deliverable:** Evidence per behavior, not claims. Re-read §Communication before drafting the user-facing report.

Re-read [`references/prove-discipline.md`](references/prove-discipline.md) before planning the pass — it owns the operational detail: the two-step execution, prep, per-scenario reports, and red flags. Core principle: **end-to-end or it didn't happen; surface every issue; claim nothing without the cited evidence (test output, screenshot, observed state) that proves it.**

**Project harness extension point (optional).** If the project declares a verification-harness skill in `docs/TESTING.md` §Prove harness, invoke that skill for the pass instead of executing inline — print `▸ pipeline-position: prove (via orchestrator)` in the invoking turn; it owns the same contract (surface everything, fix nothing, one structured report per scenario). No declaration → run the Prove pass inline per the discipline reference. The harness is never a requirement; inline is the complete default.

**On the Prove report, triage each issue by altitude before touching it:** **mechanical** (one correct answer — typo, wrong flag, missing import) → route the fix to the originating sub-mode and continue; **design-level** (more than one defensible resolution, turns on architecture/principle) → run §Off-rails, which routes the design to the deliberator. Either way, **resume Prove afterwards** — re-run the affected scenarios, and when the pass returned early on a blocker, run the verification that blocker prevented. Prove is done only when the full verification matrix passes, not when the blocker is merely cleared.

**Defer-to-BACKLOG is a user decision, not yours.** If you think a surfaced failure is out-of-scope, pre-existing, or otherwise file-and-move-on — STOP and surface it: the failure in one line + the fix-in-session alternative + your recommendation, then wait for explicit assent. Do not silently file to BACKLOG. The forbidden self-deferral/substitution rationales that signal this failure mode are enumerated in [`references/review-gates/forbidden-rationales.md`](references/review-gates/forbidden-rationales.md) — any of them surfacing in your own reasoning means stop. Once the user assents: file in `BACKLOG.md` with reproduction steps; never describe a deferred failure as working.

<HARD-GATE>
**Plan-completion gate.** Before any commit that asserts a plan is done — moving the file to `plans/shipped/`, updating INDEX `Recently shipped`, closing a gate, marking a task or row as complete in PROGRESS/FEATURES — re-read the plan's verification matrix. Confirm EVERY required outcome is in a captured verification report. Not "covered by integration tests," not "math is unit-tested," not "proven by a prior phase," not "the chain is the same as Row N which passed." Partial coverage requires explicit written user assent before commit.
</HARD-GATE>

Then: update docs per project `CLAUDE.md` §"Update docs before pushing". If installed, invoke `superpowers:finishing-a-development-branch`.

## Wrap

**Deliverable:** Session-end discipline complete.

Invoke `dev-session-wrap`. That skill owns the 5-step checklist with DoD per step (docs / commit / self-reflect / notes / memory).

The Wrap phase is triggered:
- Explicitly via `/dev-session-wrap` or "wrap up" / "end of session" user input
- Implicitly when a feature shipped or the user signals stop — you invoke wrap yourself

Do NOT skip Wrap when a feature shipped this session. The 5 steps are mandatory per project CLAUDE.
