# Self-check gates — run against your own artifact before claiming done

Run these on every spec, plan, and proof report. Read the artifact end-to-end; answer each check in your own words; any violation → rewrite that section, re-read, repeat. These summaries condense the canonical gate files under `.claude/skills/dev-orchestrator/references/review-gates/` — on any doubt, the canonical file wins.

## 1 · Triad (spec / plan / proof) — canon: `review-gates/triad.md`

- **Self-deferral.** Any section pushing off a failure / scope item / claim with the semantic intent of §4's forbidden list — intent, not literal phrases.
- **Vague confidence.** "works" / "passes" / "is correct" / "looks good" / "handles cases" / "as appropriate" without citing the specific test output, screenshot, observed state, or command that proves it.
- **Evidence gap.** Each correctness claim's evidence is (a) cited inline by name/path/output, (b) reproducible from the citation alone, (c) actually about what's claimed — not a sibling test, not unit-only when the surface under change is the handler, not unit-tested when the user traverses the UI.

## 2 · Abstraction & reuse (any introduced primitive / module / type / operation / name) — canon: `review-gates/abstraction-and-reuse.md`

Separate **invariant essence** from **per-case variation**: essence gets one home; variation sits at the edges (param / config / strategy / extension data). Reject:

1. **Near-duplicate variant** — new case mostly an existing one → reuse + lift the differing part to a parameter, not a forked copy.
2. **Per-consumer bespoke** — capability ≥2 consumers need the same (server validation + client preview; two callers of one capability) → one core primitive in a shared home + thin adapters, not fused to one caller.
3. **Abstraction leak** — a core/shared definition names a detail meaningful to only one case; push the detail to the variation layer (parameter / config / extension data). When the project declares an extension architecture, inventory the declared extension directory before naming — **adding an extension must need only its own config/data, never core edits.**

Build the shared core now when the user's direction, a few seconds' reasoning, or grounding shows ≥2 consumers will exist; otherwise (merely hypothetical consumer) YAGNI applies — but YAGNI never overrides a *stated* need. Document under `## Abstraction & reuse audit`: per name → verdict (`right-level` | `duplicates <x>` | `fuses case <x>` | `leaks <detail>`) → resolution. Any duplicates/fuses/leaks, or a missing section when shared vocab was introduced = reject.

## 3 · Matrix-row strictness (every verification-matrix row) — canon: `review-gates/matrix-row-strictness.md`

1. **File/script named** — Method names an existing path or an explicit "create at <path>" task. "scenario N", "bridge script", "audit catches X" without a path = violation.
2. **Layer tag** from the set declared in `docs/TESTING.md` §Verification layers, routed per any layer routing it declares. Wrong layer = violation.
3. **Done-check user-visible/observable** — "activity log shows line X", "panel field Y = Z", "grep returns N", "exit code 0". Not "feature works" / "tests green" / "scenario passes".
4. **Test-doesn't-exist → authoring task** earlier in the plan; the matrix can't claim unwritten coverage.
5. **No vague Behavior** — name the specific user-visible behavior/state change. "Checkout works" ✗; "Checkout produces 4 events sharing one correlationId in the event log" ✓.
6. **Invocable through real usage — never an input crafted to pass** — reachable the way a real user reaches it: the real surface, real inputs (data/config/flows that already exist) + the application state a real user needs, both in the same phase. Draw the route and inputs from what exists before constructing anything; a contrived input exercising an idealized path while the real path stays unwired = violation. A throwaway probe to pass = violation; in shipped data = shipped pollution. Probe-only exception: an error/unreachable path real usage can't express, living in test fixtures/overrides only.
7. **Asserted signal is emitted** — if a Done-check reads a log line / dialog / API event / structured record, the plan authors that emit end-to-end (registration, render entry, handler + consumer, message keys/localization per project conventions). A reference-taking operation needs a failure-path assertion: unknown/invalid reference → error signal + warning log, never a silent no-op.
**Two-step model** (rules 8–9 = step 1; rule 10 = step 2):
- **Step 1 — persisted proof (ALWAYS, but don't reduplicate):** each behavior proven once, routed to the layer `docs/TESTING.md` dictates, named for the behavior it proves. If existing tests already prove it, add NO new persisted row — cite the existing home. A row reduplicating an already-proven behavior = violation.
- **Step 2 — live verification (CONDITIONAL):** a live pass through real usage on the real surface. Where the project declares a variation axis (plugins/configs/locales — per its own docs), the variant set is **read-first-then-consult** (the motivating variant → read others → expand where the behavior is genuinely reachable → consult the user only on genuine judgment). Skipped only when nothing surfaces live (pure internals, nothing user-reachable) — skip stated with justification.

8. **Persisted rows named for the behavior they prove (step 1)** — never for the caller, surface, or variant invoking it. Litmus: if a different caller or variant would call the behavior something else, the name is wrong. (Exceptions: a row asserting a named variant's config wires the behavior — only where the project declares a variation axis; and a step-2 live row.)
9. **One persisted proof per behavior (step 1)** — a behavior decided identically everywhere is proven once; an intentional config-difference is one parameterized case inside that proof. A `<behavior>-<variantA>`/`<behavior>-<variantB>` **persisted** row pair = violation.
10. **Step 2 present, or its absence justified** — if the change altered anything user-reachable, a live row is REQUIRED (the motivating surface/variant + each scoped expansion; never the full variant list). If nothing surfaces live, step 2 is correctly absent but the matrix must **state that justification** ("step-1-only: pure internals, nothing live"). Silent step-2 omission = violation; a live row on a variant where the behavior isn't genuinely reachable = violation.

## 4 · Forbidden rationales (plan / spec / proof main body) — canon: `review-gates/forbidden-rationales.md`

Allowed only inside an explicit §Out-of-scope with a one-line justification naming the scope edge. Anywhere else = violation:

1. "Pre-existing — predates this work / out-of-scope for this row"
2. "Covered elsewhere — integration tests assert this / proven by a prior phase"
3. "Math is unit-tested"
4. "User will see it next session anyway"
5. "Plan said X but Y is close enough" (substitution without assent)
6. "Quick workaround so I can keep moving"
7. "Will file in BACKLOG later"
8. "The lower layer proves the chain — the user-visible layer is incremental"

## 5 · Placeholders (any occurrence = violation) — canon: `review-gates/placeholder.md`

`TBD` · `TODO` · `fill in` · `details to follow` · `Similar to <other>` (no concrete ref) · `Add appropriate <X>` · `handle edge cases` · `placeholder` · `implement later`
