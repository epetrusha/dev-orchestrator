---
name: dev-writing-plan
description: "Use after dev-brainstorm has produced an approved spec (state-checked, not text-trusted). Writes an implementation plan with bite-sized tasks (2-5 min each), exact file paths, DoD per task, verification matrix mapped to user-action sequences, file map split inline-vs-subagent. Saves to docs/planning/plans/active/. Do NOT load for: spec writing (use brainstorm), execution (use build), verification, session-wrap."
---

# Dev Writing-Plan

Write an implementation plan from an approved spec. Cold-start implementer: every file, every command, every test in the plan.

**Announce at start:** "Using dev-writing-plan to create the implementation plan."

**Save to:** `docs/planning/plans/active/YYYY-MM-DD-<feature-name>.md`

## Pre-conditions

Before drafting any plan content, confirm an approved spec exists under `docs/planning/specs/active/` — one the user explicitly approved in `dev-brainstorm`. No approved spec → STOP, return to `dev-brainstorm`. Do not draft a plan without one.

<HARD-GATE>
Plan document only. No code, no subagent dispatch, no implementation. Implementation begins in the orchestrator's Build phase after plan approval.
</HARD-GATE>

## Checklist (TodoWrite-tracked, in order)

1. **Re-read the spec.** Quote Intent + three-seats output into the TodoWrite closure.

2. **Scope check.** Multi-subsystem → recommend decomposing. Touches engine + UI → declare backend/frontend phase split (ideally separate sessions).

3. **File map.** Enumerate every file: exact path | inline-or-subagent | one-line responsibility. `>3 files = orchestrate` per project CLAUDE.

4. **Subagent brief discipline** (every subagent dispatch):
   - Allowed: Read, Edit, Write, Grep, Glob. Bash commands listed explicitly; `Bash: none` if not needed.
   - Forbidden: PowerShell, test runners, `git commit/push/checkout/reset/add -A/add .`, Agent, Skill.
   - Brief states what to write, never "verify by running tests".
   - Stage allowed: specific files only.
   - **Brief MUST include exact paths** for any spec, plan section, or code the subagent needs to read. No implicit context inheritance — subagents see only the orchestrator's `prompt` parameter. List paths explicitly.
   - **Briefs are AUTHORED here** (in the plan body, per task). `dev-build` dispatches as-is; the build skill does not re-author briefs.

5. **Verification matrix.** Dispatch the `dev-writing-verification` agent. The agent authors the matrix; this skill never authors it directly (writer-grades-own-homework anti-pattern). Brief inputs:

   - spec-path: the approved spec
   - plan-body-so-far: the plan content authored above this section
   - affected-files: the file map from step 3
   - applicable-plugins: plugin ids if the change introduced engine vocabulary; empty list otherwise
   - matrix-strictness-reference: `.claude/skills/dev-orchestrator/references/review-gates/matrix-row-strictness.md`

   The agent returns the matrix section as markdown. Insert it into the plan body verbatim. If the agent's return includes "does not yet exist, please add authoring task at position N" advisories, add the corresponding tasks before continuing.

   Required matrix row shape (enforced by the agent against matrix-row-strictness.md):
   ```
   | Behavior | Layer | Method | Done-check |
   ```
   Layer must be from the verification layer set declared in `docs/TESTING.md` §Verification layers. Method must name an existing file or an explicit "create file at <path>" plan task. Done-check must be user-visible or directly observable (not "tests pass").

6. **Per-plugin verification** (when spec introduced core vocabulary; applies only to projects with a declared core/extension split — see the plugin-independence gate). For every extension in the project's declared extension directory, at least one matrix row asserting the change works under that extension's config. Done-check derives from that extension's config shape. Never hardcode names not present in that directory.

7. **Tasks decomposed into 2-5-min steps.** Each step: exact paths, full code blocks (no "implement appropriately"), exact commands with expected output, `[ ]` checkbox prefix.

   - **Each Task block MUST include a `**DoD:**` line.** Yes/No verifiable. Self-review greps for `^\*\*DoD:` per Task heading; count must match Task count.

7b. **`[CHECKPOINT]` markers required**. At least one TodoWrite item per plan phase MUST contain the literal substring `[CHECKPOINT]` in its content. Standard placements: end of each plan phase, immediately before the Prove/verification pass, immediately before any commit that claims plan-completion, and after each subagent batch returns. Plain (non-checkpoint) items complete silently — use them for routine progress. **DoD grep:** `grep -nE '\[CHECKPOINT\]' <plan-path>` returns ≥1 hit per phase heading.

7c. **Engine-primitive implementation grounding** (when the spec introduces new ops, state fields, dispatchers, or event types). For each new engine primitive the plan MUST state HOW it is built, not only that it exists: (a) its data shape; (b) the existing function/seam it hooks — cited `file:line` and confirmed by reading, **including which execution path actually runs it** (pure function vs deferred/async dispatch vs a fallthrough path), not just the data shape; (c) the algorithm. A task that names a new op without its build path is incomplete. The plan inherits the spec's engine-state claims — re-verify any the design leans on; a citation carried over from the spec is not a verified citation. **DoD grep:** every new op/primitive named in the file map has a matching `**Implementation:**` block (or in-step code block) describing its seam.

7d. **Testability prerequisites** (per behavior the matrix will verify — graded by matrix-row-strictness rules 6-7). A behavior is testable only if it can be *invoked* AND *observed*; both are deliverables in the behavior's own phase, not assumptions the matrix silently makes:
   - **Invocable:** the minimal probe content that *uses* the primitive (a plugin definition/recipe, a config block, or a fixture/template) PLUS the application state that lets a real user reach it (correct permissions, required preconditions met, active session, inputs available). Authored in the behavior's phase — not deferred to a final content task.
   - **Observable on success AND failure:** the signal a test reads — a log/audit entry, a confirmation/decision dialog with a stable test id where there's a prompt, an API emit plus the client handler that consumes it, an event-log line where worth post-hoc debugging, and a structured debug/decision emit where the test must assert the *decision*, not just the end state. Author whatever registration, message keys, or localization the project's conventions require for that signal. Plus the failure path: an unknown/invalid reference emits an error signal + warning log, never a silent no-op.
   A matrix Done-check that asserts on a log line, dialog, or event the plan does not emit is unverifiable; a behavior with no probe + reachable state cannot be exercised. Author the content and the emit, not just the mutation.

8. **No placeholders.** Forbidden phrases — grep before presenting:
   ```
   TBD | TODO | fill in | implement later | details to follow | placeholder
   Add appropriate | handle edge cases | Similar to Task
   ```

9. **Out-of-scope section.** One-line justification per deferral. Copout phrases — STOP and surface for user assent:
   ```
   pre-existing | out of scope (unjustified) | covered elsewhere
   engine-proven | math is unit-tested | will file in BACKLOG later
   ```

10. **Done definition.** Matrix rows captured, audits clean, test suite green (state expected count), docs updated (PROGRESS, FEATURES, BACKLOG, INVARIANTS and the project's domain docs as applicable), plan moved to `plans/shipped/` with commit range, INDEX updated, UI screenshots in the project's screenshots dir if UI-touching.

11. **Frontmatter:**
    ```yaml
    ---
    Date: YYYY-MM-DD
    Status: Ready
    Spec: planning/specs/active/<topic>.md
    Depends-on: <other-plan-path>  # if applicable
    ---
    ```

12. **Self-review.** Fresh eyes, fix inline:

    - Spec coverage: every requirement has a task?
    - Placeholder grep (forbidden list above)
    - Copout grep (forbidden list above)
    - Name consistency across tasks (functions, methods, fields)
    - Matrix coverage: every spec behavior has a row?
    - Per-plugin rows present if applicable?
    - **DoD per task:** `grep -c '^\*\*DoD:' <plan-file>` ≥ Task heading count.
    - **Brief path completeness:** each subagent brief block (if any) contains explicit file paths for spec/plan/code references.

<HARD-GATE>
After the self-review passes, dispatch the `dev-reviewer` agent for the plan just written:

- artifact-path: the plan file path
- artifact-type: `plan`
- reviews-output-path: `docs/planning/reviews/<slug>-plan.md` (slug = plan basename)
- references-base: `.claude/skills/dev-orchestrator/references/review-gates/`

Read the resulting reviews file. The `**Status:**` line gates progress:

- `**Status:** approved` → proceed to step 13.
- `**Status:** rejected` → read cited violations, fix inline, re-dispatch. Loop until approved.

Reviewer applies gates: triad, placeholder-and-copout, matrix-row-strictness, forbidden-rationales. Arms-length neutrality is structural.
</HARD-GATE>

13. **Commit:** `docs: plan <topic>`. Update `docs/planning/INDEX.md`.

14. **Present plan for user review.** Single-content message: "Plan written and committed to `<path>`. Matrix: `<N>` rows. Files: `<count>` (`<inline>` inline, `<subagent>` subagent). Please review and approve before execution."

15. **Wait for explicit approval.** Engagement is not approval.

16. **On explicit approval, transition to dev-build.** Mark the plan `Status: Ready`.

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
