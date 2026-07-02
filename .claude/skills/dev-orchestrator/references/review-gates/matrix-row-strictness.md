# Review gate: matrix-row strictness

Every row in a verification matrix must satisfy all of the following. Any row that fails one or more is a violation.

## Required per row

1. **Test file or script named.** The Method column must name an existing file path or an explicit "create file at <path>" task earlier in the plan. Phrases like "scenario N", "scenario N PASS", "bridge integration script", or "audit catches X" without a path are violations.

2. **Layer tag.** The Layer column must be from the verification layer set declared in `docs/TESTING.md` §Verification layers. Mixed layers are allowed but each must be named. When `docs/TESTING.md` declares routing rules for which behaviors belong at which layer, a row pinned to the wrong layer is a violation.

3. **Done-check is user-visible or directly observable.** Not "tests pass". Examples of acceptable: "activity log shows line X", "the detail panel field Y equals Z", "grep returns count N", "Status: approved present", "exit code 0". Examples of violations: "feature works", "tests green", "scenario passes", "behavior is correct".

4. **Test-doesn't-yet-exist requires authoring task.** If the row references a test or script that does not yet exist, the plan must include an explicit task to author that test before the row is verified. The matrix cannot claim coverage that has not been written.

5. **No vague behavior column.** The Behavior column must name a specific user-visible behavior or state change, not a generic feature category. "Checkout works" is a violation; "Checkout produces 4 events sharing one correlationId in the event log" is acceptable.

6. **Behavior is invocable through real usage — never an input crafted to pass.** A behavior row must be reachable the way a real user reaches it: through the product's real surface, with real inputs — data, config, or flows that already exist in the product — AND the application state that lets a real user invoke it (correct permissions, preconditions met, active session, inputs available). Draw the route and inputs from what exists before constructing anything; a contrived input that exercises an idealized path while the real path stays unwired is a violation (it routes around the real seams). A throwaway `*_probe`/`*_dummy` crafted only to pass is a violation, and one written into shipped data is shipped pollution. **Probe-only exception:** a constructed input is acceptable solely for an error/unreachable path that real usage cannot express (e.g. unknown-reference → error event), and it lives in test fixtures/overrides, never shipped data. Route + reachable state are created in the same phase, not deferred.

7. **Asserted signal is emitted.** If a Done-check asserts on an emitted signal — a log/audit entry, confirmation dialog, API event/response, or structured debug/decision emit — the plan must author that emit end-to-end: the registration/render/handler work that produces the signal, including any message keys or localization the project's conventions require. A Done-check that reads a signal the plan never produces is unverifiable. Corollary: a reference-taking operation needs a failure-path assertion — an unknown/invalid reference emits an error signal + warning log, never a silent no-op — because "state mutated, nothing emitted" or "silently skipped" is the recurring escape (the silent-no-op class).

## The two-step verification model

Every feature's matrix is built from two steps. **Step 1 is invariable; step 2 is conditional.** Rules 8–9 govern step 1; rule 10 governs step 2.

- **Step 1 — persisted proof (ALWAYS; rules 8–9), but don't reduplicate.** Routed to the layer `docs/TESTING.md` dictates. **If existing tests already prove the decomposed behavior**, add **no new persisted row** — cite the existing home instead. Coverage exists for every feature; new authoring happens only where a gap is real.
- **Step 2 — live verification (CONDITIONAL; rule 10).** A live pass (UI/e2e or integration script) through **real usage (rule 6), on the real surface**. Where the project declares a **variation axis** (plugins, configs, locales, tenants — per its own docs), the variant set is **read-first-then-consult**: the variant that motivated the change always; read the others and expand where the behavior is genuinely reachable there; surface to the user only on a genuine expansion judgment.

8. **Persisted row named for the behavior it proves (step 1).** Never for the caller, surface, or variant that invokes it — a persisted row's Behavior/Method cell dramatized in one context's vocabulary is a violation. Litmus: if a different caller or variant would call the behavior something else, the name is wrong. Exceptions: a row asserting that a *named* variant's config wires the behavior (only meaningful where the project declares a variation axis), and a step-2 live row.

9. **One persisted proof per behavior (step 1).** A behavior decided identically everywhere is proven **once** — never one row per caller or variant. A behavior *intentionally different by config* is **one** parameterized case conditioned on that config — the difference lives in the data, still one row, not N. A `<behavior>-<variantA>` / `<behavior>-<variantB>` **persisted** row pair is a violation: a second, hand-maintained source of truth that drifts.

10. **Step 2 present, or its absence justified.** A live row is concrete *by design* — it names the real usage (rule 6) and the state it runs in; naming the actual surface or variant there is correct, not a violation. **If the change altered anything user-reachable, a step-2 live row is REQUIRED** — on the motivating surface/variant plus each expansion scoped read-first-then-consult (never the full variant list by default, never "all by construction"). **If the change surfaces nothing live** (pure internals), step 2 is correctly absent — but the matrix must **state that justification explicitly** ("step-1-only: pure internals, nothing user-reachable changed"). A *silent* omission of step 2 is a violation (indistinguishable from forgetting it); a step-2 row on a variant where the behavior isn't genuinely reachable is a violation (meaningless assertion).

## Output for the reviewer agent

For each violating row: cite row number + quote violating cell + rule(s) violated. Status `rejected` if any row violates.
