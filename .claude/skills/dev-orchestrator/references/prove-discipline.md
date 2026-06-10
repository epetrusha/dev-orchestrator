# Prove discipline

Operational reference for the Prove pass. The orchestrator re-reads this before planning any verification pass. A project-declared harness skill (see `docs/TESTING.md` §Prove harness) implements this same contract; this file is the default inline form.

## Core principle

End-to-end or it didn't happen. A pass is only meaningful if it traces the user's actual chain — input/click → transport → handler → state mutation → emitted signal → rendered outcome. "Tests pass" is not "feature works": the unit layer bypasses handlers, integration bypasses rendering, rendering bypasses the click flow. Pick the cheapest layer that proves the chain the user traverses, then prove the chain — not a truncated slice of it.

Verify and fix stay separate. The pass surfaces every issue; fixing happens after the report, then the affected scenarios re-run. Mid-pass fixing hides bugs: a patch lands on one symptom while the same pass surfaced four others that go unrecorded.

## Mandatory prep — produce, don't just read

Before any setup or execution:

1. Re-read `docs/TESTING.md` (layer table + declared signal conventions) and every `docs/INVARIANTS.md` topic that touches the change.
2. Check the project's interface contracts (manifests, schemas, API indexes — whatever it keeps) for any event or function under test. If a scenario asserts a payload shape or signature that differs from the contract, the scenario is wrong before it runs.
3. Read the change set — `git diff` the commits, read the plan/spec, read the affected files. Identify what specifically must work.
4. Read existing test patterns that solve the same problem class; reuse what fits. When the existing pattern is wrong-shape for this verification, the architecture-sufficiency argument below takes precedence — extend or write new helpers rather than copy forward a brittle pattern.

Then PRODUCE three outputs before any scripting (reading produces "I read it"; producing forces comprehension that reading alone does not):

- **Per-scenario assertion shape.** For each scenario: regex-strict / output-scan / state-diff snapshot / rendered-state observation / mixed-and-which. Shapes can differ across scenarios of one sweep.
- **Architecture sufficiency.** Name the tools, scripts, and helpers this pass will use; list their known limitations; argue explicitly whether they suffice as-is or a prep task must extend the harness first (new primitive, new helper). "Extend first" becomes a sub-task before scenario execution starts.
- **Applicable gotchas.** Documented gotchas from the project docs that touch this work — or an explicit "no applicable gotchas found in re-read."

A pass that skips prep produces shallow coverage and surfaces the wrong things. Prep is not overhead; it is the work.

## Plan the pass

Output: a written checklist of user-action sequences with expected outcomes — one TodoWrite item per scenario, marked completed ONLY after that scenario's report is written. A scenario without a todo entry is a scenario you can silently drop; the todo list is the force-function against silent row-skipping.

- **Map affected surfaces** from the change set; **trace each end-to-end**: who triggers it, what emits, what handler runs, what state mutates, what broadcasts, what the user sees.
- **Translate to scenarios from the user's seat**, not the code's: who runs it and from where; what pre-state must exist; what the expected visible outcome is. The same code path reached from two entry surfaces with different gating is two scenarios — success on one does not prove the other.
- **Batch setups.** One prepared environment can exercise many scenarios in sequence; don't recreate state per scenario.
- **Determinism upfront.** Seed every random or variable input for the whole sequence before acting. Don't run live randomness and reason about variance.
- **Observation beats.** One screenshot/readout per checked state — the post-action state IS the assertion. Pre-action capture only when before/after comparison is the assertion.

## Variants — cheapest first

`unit` (pure logic) → `audit` (static checks, drift/conformance scripts) → `integration` (handler flow against a running service) → `e2e/ui` (what the user clicks) → `manual` (user drives; you observe and capture). Escalate only when the cheaper layer can't prove the chain — but when the spec or acceptance criteria name a user-visible surface, the UI layer is load-bearing, not last-resort: cheaper layers prove emit shape; only the rendered surface proves rendering.

## Capture — everything, unfiltered

Pull tool output, service stdout (the unfiltered channel, not a level-filtered log file), structured event/log deltas, console errors, screenshots, state readouts. Don't filter at capture time.

**Anomaly recognition.** Anything visible in the captures that looks off — raw identifiers where names should appear, missing visual feedback, console errors during an ostensibly-passing flow, elements rendering wrong — gets logged in the report's issues section. The check is "did I see something that looked wrong?", NOT "did my assertion catch it?". Anomalies noticed-but-not-asserted-on are the single largest class of bugs that escape verification passes.

## Report — per scenario, required, never batched

No report = no pass, regardless of how obvious the outcome looked. One structured report per scenario, written before the next scenario starts. Template:

```markdown
## Scenario: <id / one-line description>   (variant: unit | audit | integration | e2e | manual)

### Plan executed
- Pre-state: <summary>
- Determinism: <seeded inputs + values>
- Observation beats: <list>

### Execution
- Steps run: <numbered, in order>
- Per-step result: <pass | fail | partial — one line each>

### Issues surfaced (ALL of them, scoped or not)
For each:
- **Where:** <file:line | test name | scenario step | URL + element>
- **What:** <one sentence>
- **Evidence:** <log excerpt | screenshot path | tool output — verbatim>
- **Scope:** <in-scope-for-current-task | out-of-scope-but-found>
- **Severity guess:** <high | med | low>
- **Downstream check done:** <what was checked, ≤5 tool calls, what it confirmed or didn't>
- **Reproduction:** <exact command or user steps>

### Captured artifacts
- <screenshots, log deltas, console errors, stdout warnings — paths/counts/samples>

### Summary
- Required outcomes confirmed: <yes | no | partial — list>
- Issues surfaced: <count>
- Recommendation: <continue | stop and fix | escalate to user>
```

Before sending, apply `review-gates/triad.md` to the report — self-deferral, vague-confidence, evidence-gap — semantically, not by regex. Then stop: the pass reports; deciding what to fix is the orchestrator-and-user's call.

## Red flags — stop and reconsider

| Red flag | What it means | Move |
|----------|---------------|------|
| "Tests passed, skip the log tail" | Optimizing for green | Tail the unfiltered output. Silent warnings are bugs |
| "Run it 3× to confirm" | Treating deterministic tests as flaky | One run is the gate. Multi-run only for a specific flake hypothesis |
| "This bug looks unrelated, skip it" | Filtering the report | Report it. The orchestrator filters, not the verifier |
| "Quick fix while I'm here" | Mid-pass fixing | Stop. Report. Hand back |
| "Screenshot of the loaded page is proof" | Loading ≠ exercising | Assert the post-action visible state |
| "I'll pivot to a cheaper layer — the planned one is blocked" | Silent layer substitution | Pivoting is allowed; *silent* substitution is not. Mark the planned layer UNVERIFIED, surface the blocker, let the orchestrator decide |
| "Pre-existing / covered elsewhere / math is unit-tested" | Self-deferral language | Forbidden in pass output. Report the gap; deferral is decided above |
| "I'll fold this into the next scenario's report" | Report batching | Forbidden. Per-scenario, before the next begins |
| "Skip the report — the result is obvious" | Report skipping | Forbidden. No report = no pass |
| "I noticed X but it wasn't my assertion" | Anomaly filtering | Report it. See §Capture |
| "I'll deep-dive this root cause now" | Mid-pass scope creep | ≤5 tool calls per surfaced issue; note "diagnosis incomplete" and continue |

## Efficiency directives

- One setup, many actions. One log snapshot, many scenarios — diff at the end.
- Deterministic over flake-tolerant, always.
- Use the cheapest reader for state checks (test-context mirror, API readout); reserve the expensive surface (browser, full UI) for what only it can answer.
- Combine no-service variants back-to-back; share one running service across integration and UI passes.
