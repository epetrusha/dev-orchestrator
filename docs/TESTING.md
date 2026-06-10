# Testing

Verification posture for this project. The dev-* skills read this during verification work before picking a test layer.

## Principle

End-to-end or it didn't happen. Pick the cheapest layer that proves the user's actual chain: unit tests for pure logic, integration/API tests for handler flow, a UI/end-to-end pass for anything the user clicks. Write the integration script first and make it pass green before the feature commit lands.

## Verification layers

The layer vocabulary used by verification-matrix rows (`dev-writing-plan` step 5, `matrix-row-strictness` rule 2). Default set:

`engine` (core logic) · `api` (handler/request flow) · `ui` (what the user sees/clicks) · `audit` (logs/event records) · `fs` (files on disk) · `content` (authored data/config) · `exec` (script/CLI runs) · `bootstrap` (startup/load path)

Adapt this set to your stack — e.g. swap `engine`/`content` for `db`, `queue`, `cli` — and keep this section as the single declaration; the matrix author and reviewer validate rows against the set declared here.

## Prove harness

Default: **none** — the orchestrator runs the Prove pass inline per `.claude/skills/dev-orchestrator/references/prove-discipline.md`. A project with its own verification-harness skill declares it here by name; the orchestrator then invokes that skill at the Prove phase under the same contract (surface everything, fix nothing, one structured report per scenario). Declaring a harness is optional — inline is the complete default.

## What to use when

_Add project-specific test layers, harnesses, and helpers here as the codebase grows._
