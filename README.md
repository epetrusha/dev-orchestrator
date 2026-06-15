# dev-orchestrator

A reusable AI-development workflow for Claude Code: the `dev-*` skill suite plus the project conventions that pair with it. Drop the skills and `CLAUDE.md` into a project to enforce an architect-auditor pipeline — **brainstorm → plan → build → prove → wrap** — where every phase is gated and every behavior is proven before it's claimed.

This is the heavyweight option. It trades tokens and turns for rigor, and it's built for people who have been burned by "tests pass" that didn't mean "it works."

## Why it exists

Agentic coding fails in recurring, recognizable ways:

- **Premature completion claims** — "done" backed by a passing unit test while the user-facing chain was never exercised.
- **Self-graded homework** — the same context that authored a plan reviews it and finds it excellent.
- **Hallucinated codebase state** — "the engine already supports X," sometimes even with a real-looking `file:line` citation that misdescribes what the code does.
- **Silent scope drift** — a planned verification quietly substituted for a cheaper one mid-execution.
- **Copout deferral** — failures filed away as "pre-existing" or "covered elsewhere" without the user ever deciding that.

The suite converts each of these from an exhortation ("be careful") into structure. Every mechanism below exists because the failure it counters actually happened.

## The pipeline

```text
mode gate ──► investigate ──► scope ──► brainstorm ──► plan ──► build ──► prove ──► wrap
 (classify     (read-only      (user     (spec +        (plan +   (todo-    (evidence  (5-step
  the work      diagnosis)      gate)     reviewer       reviewer  tracked    per         close-out)
  out loud)                              gate)          + matrix  dispatch)  behavior)
                                                         agent)
```

Hard gates between phases: no spec without scope approval, no plan without spec approval, no code without plan approval, no completion claim without captured evidence per verification-matrix row. Engagement is not approval — only an explicit yes clears a gate. Any mid-execution deviation from the approved plan stops the work and goes back to the user.

## What's distinctive

### Arms-length review by construction

Specs and plans are reviewed by a fresh-context subagent (`dev-reviewer`) that applies shared review-gate files and returns a written verdict — reject means fix and re-dispatch until approved. The verification matrix is authored by a second independent agent (`dev-writing-verification`), because a plan author grading its own verification rows is the writer-grades-own-homework anti-pattern. Neutrality isn't a prompt instruction; it's structural — the reviewer has no access to the authoring context that rationalized the shortcuts.

The gates themselves (`triad`, `placeholder`, `matrix-row-strictness`, `forbidden-rationales`, `abstraction-and-reuse`) are standalone vocabulary files consumed by both the authoring skills and the reviewer — one canonical statement of each rule, no drift between what the author checks and what the reviewer enforces.

### Proof, not claims

The Prove pass is its own phase with its own discipline: pick the cheapest layer that exercises the user's *actual chain* end-to-end, one report per scenario (batching hides bugs — one fix lands while four other failures go unrecorded), verify-then-fix kept separate, and a plan-completion gate that re-reads the verification matrix before any "done" commit.

Verification-matrix rows are held to strictness rules that make hollow coverage structurally hard: every row names a real test file or an authoring task for one; every done-check is user-visible or directly observable ("activity log shows X", never "tests pass"); every behavior must be **invocable** (the plan authors the probe content and the application state that lets a real user reach it) and **observable** (the plan authors the emitted signal the check reads — on success *and* failure; "state mutated, nothing emitted" doesn't pass review).

A list of **forbidden rationales** ("pre-existing", "covered elsewhere", "math is unit-tested", "will file in BACKLOG later") acts as tripwires: if one appears in the agent's own reasoning, it must stop and surface the decision instead of self-deferring. Deferral is a user decision, never the agent's.

The pass itself follows a written discipline reference: prep that *produces* outputs (per-scenario assertion shapes, an architecture-sufficiency argument) rather than just "I read it," anomaly recognition beyond planned assertions ("did I see something off?", not "did my assertion catch it?"), a per-scenario report template, and a red-flags table. Projects with their own verification harness can declare it in `docs/TESTING.md` and the orchestrator dispatches to it under the same contract — optional; inline is the complete default.

### Grounding gates

Claims are cheap and the suite prices them accordingly. Doc reading requires **quote evidence** — a load-bearing line cited per document, not "I read it." Claims about existing code require `file:line` citations, *and* the spec self-review audits them for the subtler failure: a citation that's real but wrong about what the code does. External systems (an API, a standard, a game ruleset) get a ground check against canonical sources before design starts — never from the user's paraphrase or the model's memory. And questions the codebase can answer are answered by grep before they cost a user turn.

### Metacognitive guardrails

- **Priming anti-patterns** — at session and brainstorm entry, the agent self-checks against a catalog of named failure modes (anchoring on input wording, familiarity bias, mirroring user proposals as approval, reality-check skipped…) and states signal + countermove for each one that applies. Chat-visible, so the discipline is auditable.
- **Mental-models Challenge step** — a 40-model catalog (two volumes: systems thinking; abstraction & extensibility) from which the 5–7 most load-bearing models are applied to each task *before* design, with the application written into the spec. The point isn't the catalog — it's forcing a structurally different framing before the obvious-feeling one wins by default.
- **Degradation prevention** — tripwires for the recognizable decay modes of long sessions: same thing failing twice, describing a failure positively, easy-pass criteria, suggesting the session should end. Each triggers a stop-and-reread of the cognitive-discipline reference.
- **Lived-path self-check** — before any design is presented: walk one real use end-to-end (who acts, what flows, what they see), and let the walk redesign the feature rather than confirm it.

### Cross-session memory

Work survives the session that did it. The planning lifecycle (`specs/`, `plans/`, `investigations/`, `reviews/`, `handoffs/`, `INDEX.md`) is an auditable trail where folder placement is authoritative and a paused plan is executable by a cold-start successor. On top of that, a **knowledge base** (`docs/knowledge/`) captures the layer git history can't:

- `session-history.md` — dense per-session audit entries: what shipped (with commit hashes), corrections received, what failed and what worked, next-session entry point. Honest by convention — sanitized entries teach nothing.
- `learnings.md` — generalized principles distilled from self-reflection, held to litmus tests (would it help in a different domain? does it survive deleting every session-specific noun?) so session-overfit diary lines don't masquerade as lessons.

Session wrap is itself a gated 5-step skill (docs, commit, self-reflect, knowledge base, rule persistence) with a definition-of-done per step — not an honor-system "remember to wrap up."

## Layout

```text
.claude/
├── skills/
│   ├── dev-orchestrator/   — entry point: mode gate + phase chaining
│   │   └── references/     — communication, cognitive-discipline, mental models,
│   │                         priming anti-patterns, review-gates/
│   ├── dev-investigate/    — read-only diagnosis
│   ├── dev-brainstorm/     — idea → approved spec
│   ├── dev-writing-plan/   — approved spec → executable plan
│   ├── dev-build/          — approved plan → code
│   └── dev-session-wrap/   — session-end discipline
└── agents/
    ├── dev-reviewer.md             — arms-length spec/plan review verdicts
    └── dev-writing-verification.md — arms-length verification-matrix author
docs/
├── PROGRESS.md · FEATURES.md · BACKLOG.md · INVARIANTS.md · GLOSSARY.md · TESTING.md
├── knowledge/              — session-history.md + learnings.md (+ format primer)
└── planning/               — INDEX.md + specs/ plans/ investigations/ reviews/ handoffs/
CLAUDE.md                   — project rules the skills reference
```

## Install

1. Copy `.claude/` (skills + agents), `CLAUDE.md`, and the `docs/` scaffold into your project root.
2. Adapt `CLAUDE.md`'s Documentation map to your project's domain docs.
3. Declare your verification layer set in `docs/TESTING.md` §Verification layers (a default ships there).
4. If your project has a core/extension split (plugins, providers, themes), declare the extension directory in `docs/GLOSSARY.md`. No split → skip this step.
5. Invoke the `dev-orchestrator` skill before any implementation task. It classifies the work and chains the subskills.

See [`CLAUDE.md`](CLAUDE.md) for the full rule set and [`docs/planning/INDEX.md`](docs/planning/INDEX.md) to orient at the start of a session.

## What to customize per project

| Knob | Where | Default |
|---|---|---|
| Verification layer vocabulary | `docs/TESTING.md` §Verification layers | `engine, api, ui, audit, fs, content, exec, bootstrap` |
| Domain doc set the brainstorm reads | `CLAUDE.md` §Documentation map | INVARIANTS / TESTING / BACKLOG / GLOSSARY + your domain docs |
| Core/extension declaration | `docs/GLOSSARY.md` | none (plugin gates close as N/A) |
| Knowledge base location (session history + learnings) | `CLAUDE.md` §Knowledge base | in-project `docs/knowledge/`; point at an external vault or set `none` |
| Prove harness skill | `docs/TESTING.md` §Prove harness | none — inline Prove pass; declare a project harness skill to dispatch to it |
| Reasoning discipline references | `.claude/skills/dev-orchestrator/references/` | shipped; your global `~/.claude/CLAUDE.md` layers on top |

## Cost caveat

The full pipeline per non-trivial change is: priming → mental models → scope gate → spec → reviewer loop → plan → matrix agent → reviewer loop → build → per-scenario prove → wrap. That is a lot of tokens and user turns by design. Inline bypasses exist for 1–3 file fixes and admin work; everything bigger pays the toll. If you want a lighter-weight workflow, this isn't it.

## Origin

Started as a fork of [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent, then diverged substantially: the structural-neutrality review architecture, review-gate vocabulary, verification-matrix strictness, metacognitive guardrails, and planning/knowledge lifecycle are original to this project. MIT licensed; see [LICENSE](LICENSE) for both notices.
