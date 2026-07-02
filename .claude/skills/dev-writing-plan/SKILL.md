---
name: dev-writing-plan
description: "Use after dev-brainstorm has produced an approved spec (state-checked, not text-trusted). Writes an implementation plan with bite-sized tasks (2-5 min each), exact file paths, DoD per task, verification matrix mapped to user-action sequences, file map split inline-vs-subagent. Saves to docs/planning/plans/active/. Do NOT load for: spec writing (use brainstorm), execution (use build), verification, session-wrap."
---

# Dev Writing-Plan

Write an implementation plan from an approved spec. Cold-start implementer: every file, every command, every test in the plan.

<ENTRY-GUARD>
Run only if `▸ pipeline-position: plan (via orchestrator)` was printed above this invocation in the current session. Absent → you were reached outside the orchestrator; stop and hand back to it. You are never an entry point — the orchestrator is the only door.
</ENTRY-GUARD>

**Announce at start:** "Using dev-writing-plan to create the implementation plan."

**Save to:** `docs/planning/plans/active/YYYY-MM-DD-<feature-name>.md`

## Pre-conditions

Before drafting any plan content, confirm an approved spec exists under `docs/planning/specs/active/` — one the user explicitly approved in `dev-brainstorm` (or the governing charter, for charter-governed sub-work with a deliberation note). No approved spec → STOP, return to `dev-brainstorm`. Do not draft a plan without one.

<HARD-GATE>
Plan document only. No code, no subagent dispatch, no implementation. Implementation begins in the orchestrator's Build phase after plan approval.
</HARD-GATE>

**Design surfaced during planning** (scope cut / structural fork / new abstraction / which-of-N migrates)? STOP — a fork is the deliberator's, never settled in plan prose. Hand back to `dev-orchestrator`; §Off-rails routes it to `dev-deliberator`, then resume from the resolved shape.

## Checklist (TodoWrite-tracked, in order)

1. **Re-read the spec.** Quote Intent + three-seats output into the TodoWrite closure.

2. **Scope check.** Multi-subsystem → recommend decomposing. Touches backend + UI → declare a backend/frontend phase split (ideally separate sessions).

3. **File map.** Enumerate every file: exact path | inline-or-subagent | model tier (subagent only) | one-line responsibility. `>3 files = orchestrate` per project CLAUDE. **Model tier: default `sonnet`; reserve `opus` for design/architecture or genuinely hard reasoning — and prefer doing that inline. Never default to `opus`. `haiku` is forbidden — too unreliable for file edits.** This file map is the authoritative orchestration declaration — Build loads it, doesn't re-derive.

4. **Wiring trace — ground the design against real code BEFORE briefs, tasks, and matrix.** The deliberator had no code tools — verify the design against actual files before the briefs, tasks, and matrix (steps 5–7) are built on it. Per new mechanism the file map introduces (its `responsibility` column is the *what*; this is the *how* — don't re-list files, annotate them):
   - **Seam + execution path** (any new behavior-bearing addition — function, state field, interface, event): data shape, the function/seam it hooks (`file:line`, confirmed by reading), which execution path actually runs it, and the algorithm. Re-verify any spec claim about current code the design leans on — an inherited citation is not a verified one. **DoD grep:** every new mechanism in the file map has a matching `**Implementation:**` block.
   - **Testing home + dedup** (per mechanism — read `docs/TESTING.md` this pass): **first check whether existing tests already prove it — if so, cite that home and route no new proof** (dedup). Otherwise route it to the cheapest layer of the project's declared verification layer set that actually proves the mechanism — `docs/TESTING.md` owns the layers and the routing rules; don't invent a taxonomy it doesn't declare. One proof per behavior, named for the behavior it proves — never for the caller or variant invoking it; an input/config difference that genuinely varies is a parameterized case inside that proof, not a copied sibling test per variant. Fold into the existing test that exercises the same machinery rather than adding a sibling. Cite the project's test helpers reused rather than hand-rolled. A proof guarding only a completed migration → drop.
   - **Reachability** (when live-reachable): if the change alters anything user-reachable, name how the live pass reaches it — the **real usage** exercised (the real surface and real inputs drawn from existing data/config/flows, never crafted to pass; matrix-row-strictness rule 6), the application state that makes it reachable, and the success/failure signal (structured log/audit entry, UI surface + stable test id, API emit + consuming handler; failure → error signal + warning log, never a silent no-op). **Where the project declares a variation axis** (plugins, configs, locales, tenants — per its own docs), scope the live pass **read-first-then-consult**: the variant that motivated the change always; expand only where the behavior is genuinely reachable in a variant; consult the user only on a genuine expansion judgment. No declared axis → one live pass on the real surface. UI/UX changes get a live pass asserting full click-through, not render-only. **Nothing live-reachable** (pure internals)? No live pass — state that justification, don't silently omit.
   - **Collateral + reuse** (every task): name what implementing it requires beyond the headline deliverable — what it must touch, whether what it adds already exists (don't re-create it), the existing helper to **reuse** rather than hand-roll. A requirement discovered at execution is a planning miss — bake it into the task's build-note here.

5. **Subagent brief discipline** (every subagent dispatch):
   - Tool grants stated explicitly per task — what's allowed, what's forbidden — per the project's dispatch policy (project `CLAUDE.md` §Subagent dispatch discipline); grant what THIS task needs, nothing more.
   - Subagents implement and return; the orchestrator verifies and commits. Brief states what to write, never "verify by running tests".
   - Stage allowed: specific files only.
   - **Brief MUST include exact paths** for any spec, plan section, or code the subagent needs to read. No implicit context inheritance — subagents see only the orchestrator's `prompt` parameter. List paths explicitly.
   - **Briefs are AUTHORED here** (in the plan body, per task). `dev-build` dispatches as-is; the build skill does not re-author briefs.

6. **Tasks decomposed into 2-5-min steps.** Each step: exact paths, full code blocks (no "implement appropriately"), exact commands with expected output, `[ ]` checkbox prefix. Authored *before* the matrix so the matrix grades a complete plan and its rows cite real task numbers.

   - **Each Task block MUST include a `**DoD:**` line.** Yes/No verifiable. Self-review greps for `^\*\*DoD:` per Task heading; count must match Task count.

7. **Verification matrix — RECORDS the step-4 decisions, authored LAST over the complete plan.** Dispatch the `dev-writing-verification` agent once the file map, wiring trace, briefs, and tasks (steps 3–6) are all written, so each row's Method cites a real task number and each done-check maps to what a task actually builds. The matrix is the *record* of the testing + reachability decisions made in step 4, not a fresh judgment — the agent authors it (this skill never does: writer-grades-own-homework anti-pattern). Brief inputs:

   - spec-path: the approved spec
   - plan-body: the **complete** plan above this section — file map, **step-4 wiring trace** (grounded seams + testing homes/dedup + reachability decisions, consumed not re-derived), briefs, and the **step-6 task list** (rows cite these task numbers)
   - affected-files: the file map from step 3
   - variation-scope: the step-4 read-first-then-consult scope over the project's declared variation axis — the variant that motivated the change plus any expansion where the behavior is genuinely reachable; empty if the project declares no axis or the change is single-surface. Never the full variant list by default
   - matrix-strictness-reference: `.claude/skills/dev-orchestrator/references/review-gates/matrix-row-strictness.md` (the canonical rules — persisted rows behavior-named and deduped, live rows through real usage, skip-with-justification)
   - testing-doctrine-reference: `docs/TESTING.md`

   The agent returns the matrix section as markdown. Insert it into the plan body verbatim. If the agent flags a behavior whose proving task is missing, **add the task back in step 6** (the task list is already drafted — insert, don't forward-reference) and re-dispatch.

   Required matrix row shape (enforced by the agent against matrix-row-strictness.md):
   ```
   | Behavior | Layer | Method | Done-check |
   ```
   Layer must be from the verification layer set declared in `docs/TESTING.md` §Verification layers. Method must name an existing file or a step-6 task. Done-check must be user-visible or directly observable (not "tests pass").

8. **`[CHECKPOINT]` markers required.** At least one TodoWrite item per plan phase MUST contain the literal substring `[CHECKPOINT]` in its content. Standard placements: end of each plan phase, immediately before the Prove/verification pass, immediately before any commit that claims plan-completion, and after each subagent batch returns. Plain (non-checkpoint) items complete silently — use them for routine progress. **DoD grep:** `grep -nE '\[CHECKPOINT\]' <plan-path>` returns ≥1 hit per phase heading.

9. **No placeholders.** Forbidden phrases — grep before presenting:
   ```
   TBD | TODO | fill in | implement later | details to follow | placeholder
   Add appropriate | handle edge cases | Similar to Task
   ```

10. **Out-of-scope section.** One-line justification per deferral. Copout phrases — STOP and surface for user assent:
   ```
   pre-existing | out of scope (unjustified) | covered elsewhere
   proven by a prior phase | math is unit-tested | will file in BACKLOG later
   ```

11. **Done definition.** Matrix rows captured, audits clean, test suite green (state expected count), docs updated (PROGRESS, FEATURES, BACKLOG, INVARIANTS and the project's domain docs as applicable), plan moved to `plans/shipped/` with commit range, INDEX updated, UI screenshots in the project's screenshots dir if UI-touching.

12. **Frontmatter:**
    ```yaml
    ---
    Date: YYYY-MM-DD
    Status: Ready
    Spec: planning/specs/active/<topic>.md
    Depends-on: <other-plan-path>  # if applicable
    ---
    ```

13. **Self-review.** Fresh eyes, fix inline:

    - Spec coverage: every requirement has a task?
    - Placeholder grep (forbidden list above)
    - Copout grep (forbidden list above)
    - Name consistency across tasks (functions, methods, fields)
    - Matrix coverage: every spec behavior has a row?
    - Matrix rows conform to matrix-row-strictness (one persisted proof per behavior, deduped against existing tests, behavior-named), at the layers the step-4 testing-home decisions routed them to?
    - Live reachability pass present where the change is live-reachable, or its absence justified (step-4 reachability decision)?
    - **DoD per task:** `grep -c '^\*\*DoD:' <plan-file>` ≥ Task heading count.
    - **Brief path completeness:** each subagent brief block (if any) contains explicit file paths for spec/plan/code references.

<HARD-GATE>
After the self-review passes, dispatch the `dev-reviewer` agent for the plan just written:

- artifact-path: the plan file path
- artifact-type: `plan`
- reviews-output-path: `docs/planning/reviews/<slug>-plan.md` (slug = plan basename)
- references-base: `.claude/skills/dev-orchestrator/references/review-gates/`

Read the resulting reviews file. The `**Status:**` line gates progress:

- `**Status:** approved` → proceed to step 14.
- `**Status:** rejected` → read cited violations, fix inline, re-dispatch. Loop until approved.

Reviewer applies gates: triad, placeholder, matrix-row-strictness, forbidden-rationales. Arms-length neutrality is structural.
</HARD-GATE>

14. **Commit:** `docs: plan <topic>`. Update `docs/planning/INDEX.md`.

15. **Present plan for user review.** Single-content message: "Plan written and committed to `<path>`. Matrix: `<N>` rows. Files: `<count>` (`<inline>` inline, `<subagent>` subagent). Please review and approve before execution."

16. **Wait for explicit approval.** Engagement is not approval.

17. **On explicit approval, transition to dev-build.** Mark the plan `Status: Ready`.

## Task template

````markdown
### Task N: <Component>

**Files:**
- Create: `<exact/path/new_file>`
- Modify: `<exact/path/existing>:<line-range>`
- Test: `<exact/path/test_file>`

- [ ] **Step 1: Write the failing test**

```
# illustrative — use the project's language + test framework
from <module> import thing_under_test

def test_does_specific_behavior():
    assert thing_under_test(input) == expected
```

- [ ] **Step 2: Run, expect fail**

Run: `<test command — e.g. pytest <test-path>>`
Expected: FAIL with `<expected error>`

- [ ] **Step 3: Implement**

```
def thing_under_test(input):
    ...  # exact implementation
```

- [ ] **Step 4: Run, expect pass**

Run: `<test command — e.g. pytest <test-path>>`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add <files>
git commit -m "feat: <message>"
```
````

## Cross-cutting rules

Communication discipline, engagement-vs-approval, hard-correction handling, length ceilings, jargon discipline — defined in `dev-orchestrator`. Re-read that block before every user-facing message.
