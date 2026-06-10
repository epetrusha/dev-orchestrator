# Testing

Verification posture for this project. The dev-* skills read this during verification work before picking a test layer.

## Principle

End-to-end or it didn't happen. Pick the cheapest layer that proves the user's actual chain: unit tests for pure logic, integration/API tests for handler flow, a UI/end-to-end pass for anything the user clicks. Write the integration script first and make it pass green before the feature commit lands.

## Verification layers

The layer vocabulary used by verification-matrix rows (`dev-writing-plan` step 5, `matrix-row-strictness` rule 2). Default set:

`engine` (core logic) · `api` (handler/request flow) · `ui` (what the user sees/clicks) · `audit` (logs/event records) · `fs` (files on disk) · `content` (authored data/config) · `exec` (script/CLI runs) · `bootstrap` (startup/load path)

Adapt this set to your stack — e.g. swap `engine`/`content` for `db`, `queue`, `cli` — and keep this section as the single declaration; the matrix author and reviewer validate rows against the set declared here.

## What to use when

_Add project-specific test layers, harnesses, and helpers here as the codebase grows._
