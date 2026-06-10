# Review gate: matrix-row strictness

Every row in a verification matrix must satisfy all of the following. Any row that fails one or more is a violation.

## Required per row

1. **Test file or script named.** The Method column must name an existing file path or an explicit "create file at <path>" task earlier in the plan. Phrases like "scenario N", "scenario N PASS", "bridge integration script", or "audit catches X" without a path are violations.

2. **Layer tag.** The Layer column must be from the verification layer set declared in `docs/TESTING.md` §Verification layers. Mixed layers are allowed but each must be named.

3. **Done-check is user-visible or directly observable.** Not "tests pass". Examples of acceptable: "activity log shows line X", "the detail panel field Y equals Z", "grep returns count N", "Status: approved present", "exit code 0". Examples of violations: "feature works", "tests green", "scenario passes", "behavior is correct".

4. **Test-doesn't-yet-exist requires authoring task.** If the row references a test or script that does not yet exist, the plan must include an explicit task to author that test before the row is verified. The matrix cannot claim coverage that has not been written.

5. **No vague behavior column.** The Behavior column must name a specific user-visible behavior or state change, not a generic feature category. "Checkout works" is a violation; "Checkout produces 4 events sharing one correlationId in the event log" is acceptable.

6. **Behavior is invocable.** A behavior row must be reachable by something the plan authors: probe content that uses the primitive (a plugin definition/recipe, a config block, or a fixture/template) AND the application state that lets a real user invoke it (correct permissions, required preconditions met, active session, inputs available) — both created in the same phase, not deferred to a final content task. A row asserting a behavior with no task authoring its probe content + reachable state is a violation: the behavior cannot be exercised, so the row cannot pass.

7. **Asserted signal is emitted.** If a Done-check asserts on an emitted signal — a log/audit entry, confirmation dialog, API event/response, or structured debug/decision emit — the plan must author that emit end-to-end: the registration/render/handler work that produces the signal, including any message keys or localization the project's conventions require. A Done-check that reads a signal the plan never produces is unverifiable. Corollary: a reference-taking operation needs a failure-path assertion — an unknown/invalid reference emits an error signal + warning log, never a silent no-op — because "state mutated, nothing emitted" or "silently skipped" is the recurring escape (the silent-no-op class).

## Output for the reviewer agent

For each violating row: cite row number + quote violating cell + rule(s) violated. Status `rejected` if any row violates.
