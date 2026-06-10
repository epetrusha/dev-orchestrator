# Glossary

Domain terms and what they mean in this project.

- **Architect-auditor** — the operating role the dev-* skills enforce: you design, review, verify, and decide; you own conceptual integrity rather than just making code compile.
- **Orchestration** — dispatching subagents to do the work while you review the diffs; required for change sets >3 files, multi-step plans, or mechanical work across many locations.
- **Inline work** — doing the work yourself: surgical 1–3 file fixes you already understand, one-shot edits, admin/docs/close-out.
- **Spec** — a design document in `docs/planning/specs/` capturing intent, three-seats output, and acceptance criteria; produced by `dev-brainstorm`.
- **Plan** — the executable, cold-start implementer's task list in `docs/planning/plans/`, derived from an approved spec; produced by `dev-writing-plan`.
- **Handoff** — an append-only session note in `docs/planning/handoffs/` written mid-implementation so the next session can resume.
- **Extension** — a data-driven module layered on a generic core: a plugin, ruleset, provider, theme, or tenant. If the project has them, declare the extension directory and its definition-file shapes here or in `CLAUDE.md`; the plugin-independence review gate and per-plugin verification rows key off that declaration. Projects without extensions close those gates as explicit N/A.
