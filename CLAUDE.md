# Dev Orchestrator — AI Development Context

> Project-specific rules only. The reasoning and communication discipline the skills re-read lives in `.claude/skills/dev-orchestrator/references/` (cognitive-discipline, communication, dev-mental-models) so it resolves in any installation; your own global `~/.claude/CLAUDE.md`, if you keep one, layers on top.

## What this repo is

This repo houses the **dev-orchestrator** skill system — the `.claude/skills/dev-*` skills that enforce the architect-auditor workflow (brainstorm → plan → build → prove → wrap). The skills are the operational entry point; this file holds the general project rules they reference. It is meant to be reused: drop these skills + this `CLAUDE.md` into a project, then layer project-specific docs on top.

## Entry point

**Before any implementation task — features, fixes, refactors, data-shape changes, test writing — invoke the `dev-orchestrator` skill.** It classifies the work and chains the subskills. Its HARD-GATE: no file edits, code, or subagent dispatch until the Plan phase output is explicitly approved.

The pipeline (each phase is a skill the orchestrator invokes):

- `dev-investigate` — read-only diagnosis when the request is vague or scope is unclear.
- `dev-brainstorm` — idea → approved spec in `docs/planning/specs/active/`.
- `dev-writing-plan` — approved spec → executable plan in `docs/planning/plans/active/`.
- `dev-build` — approved plan → code, dispatching the plan's authored subagent briefs.
- `dev-session-wrap` — session-end discipline.

The skills own the operational detail (checklists, review-gate dispatch, spec/plan templates, the priming and mental-model passes). This file does not duplicate it — it states the project-level rules the skills point back to.

## Operating mode

You are the **architect-auditor**. You design, review, verify, decide. You own conceptual integrity — does this serve the system, not just compile.

### When to orchestrate vs stay inline

**Inline** (you do the work): surgical 1–3 file fixes you already understand, one-shot edits, admin / docs / close-out work.

**Orchestrate** (subagents do the work, you review): any change set spanning more than 3 files, multi-step plans, mechanical work across many locations. If you're about to touch a 4th file, stop and orchestrate — don't rationalize each edit as "just one more small change." Once orchestrating, stay at that layer — don't drop into inline code mid-plan.

### Subagent dispatch discipline

Every Agent call states what tools the subagent may and may not use. Think about what THIS task needs — don't paste a generic template.

- **Always allowed:** Read, Edit, Write, Grep, Glob.
- **Bash — decide per task:** list the allowed commands explicitly; `Bash: none` when not needed.
- **Subagents do not run test runners.** They implement and return; the orchestrator runs tests with retry awareness for environmental flakes.
- **Briefs state what to write, never "verify by running tests."** That incentivizes destructive diagnostic loops on environmental flakes.
- **Briefs include exact paths** for any spec, plan section, or code the subagent must read — subagents see only the `prompt` parameter, nothing is inherited.
- **Always forbidden for subagents:** any shell other than the declared one, test runners, `git commit`, `git push`, `git checkout`, `git reset`, `git add -A`, `git add .`, Agent, Skill.
- **Shell policy (declare per environment):** pick one canonical shell for the project and name it here; orchestrator and subagents use only that shell. (This repo's author declares Bash — on Windows, PowerShell quirks made it the non-default.)
- **One command per call. No `&&`, no `;`, no pipe chains.** Permission allowlists and command-guard hooks match single commands; a chain either re-prompts the user or evades the guard. Relax only if you run fully auto-approved with no command hooks.
- **No complex one-liners.** If a command needs quoting gymnastics, subshells, or inline logic — write a script file and run it.

The orchestrator reviews diffs and commits. Subagents may stage specific files only.

### Execution-mode discipline

**Preparation is more important than execution.** Reading, planning, organizing, and declaring approach is not overhead — it is the work. Code written without preparation is code that will be rewritten.

**Pre-execution gate.** Before the first code change after plan approval: (1) load todos from the plan, (2) declare orchestration vs inline per task, (3) state model tier per subagent. No code until this is done.

**Plans will change.** No plan survives contact with reality. Track progress against the plan and modify it as new information surfaces. The plan is a living document, not a checklist to race through.

**Every ~10 tool calls during execution:** state in one sentence what you're doing and whether it matches the plan. If you can't say it clearly, stop.

### Degradation prevention

When you notice any of these, stop executing. Re-read `.claude/skills/dev-orchestrator/references/cognitive-discipline.md`. Then step up one level — question the frame, not the code.

- **Same thing fails twice** → the approach is wrong. Diagnose the assumption, don't patch the symptom.
- **10+ tool calls without self-assessment** → pause. State the goal in one sentence. Is what you're doing serving it?
- **Corrected on an approach** → abandon fully and replace. Do not iterate on the bad direction.
- **Describing a failure positively** → you're optimizing for appearance. State what actually happened.
- **Copouts, deferrals, or easy-pass criteria** → assume every output will be reviewed and judged. Shortcuts, fake tests, self-congratulatory summaries, and "good enough" declarations will be rejected. If you wouldn't defend it under scrutiny, don't ship it.
- **Declaring a feature "works" without end-to-end proof** → a feature that passes unit tests but not actual usage is a feature that doesn't work. Test the real thing: actual user actions, actual state mutations confirmed in actual outputs. "Tests pass" is not "it works."
- **Suggesting the session should end** → you are not tired. Context usage is the constraint.

## Hard gates

The dev-* skills enforce these mechanically; this is the canonical statement of intent.

0. **Prepare before planning.** Challenge the decomposition before writing a plan — look at the problem from the user's seat, the data-flow seat, and the abstraction seat. If your first framing produces a list of similar items, you're at the wrong level — find the pattern.

1. **Read docs before implementation.** Read the relevant project docs and quote the load-bearing lines before touching code. The criterion is the cheapest layer that proves the contract the user traverses. If no doc section applies, say so.

2. **Propose → Approve → Execute.** Non-trivial changes — new files, refactors, features, data-shape changes, interface changes, planning/spec lifecycle changes — require a separate proposal pass ending in an explicit question. Execution starts only on clear assent. User questions/corrections are iterations, not approval.

3. **Design checkpoint before proposing.** Redesign before presenting if: it encodes a specific case where a generic pattern exists; you have not considered a structurally different approach; you have not traced the full data flow; or the actual behavior cannot be live-verified.

4. **Plan self-check before presenting.** Does the plan pass the design checkpoint? Do any decisions defer, scope out, or take the easy path without researching the true problem boundary? Do the completion criteria map to real user-action sequences, or are they easy to pass without proving the feature works?

5. **Live-verify before commit.** Pick the cheapest layer that proves the user's actual chain end-to-end. List the sequences to exercise, write the integration script first, and make it pass green before the feature commit lands. Exercise the actual new behavior, not an adjacent path. Tests/builds/subagent reports are supporting evidence, not verification.

6. **Update docs before pushing.** Update affected docs (`PROGRESS.md`, `FEATURES.md`, `BACKLOG.md`, planning files). Add a session entry to `PROGRESS.md`; move shipped items to `FEATURES.md`; update planning `INDEX.md` when files move.

## Execution discipline

After subagent work, read the diff, not the report. `git diff`, grep expected symbols/consumers, trace integration across the layers.

Before writing new code (test scripts, utilities, integrations), read existing files that solve the same class of problem. Import the existing pattern; don't reimplement it.

One commit per coherent feature/fix. Before fixing anything, state: `User wants X; current code fails because Y; I will change Z.` If you cannot say it clearly, do not code yet.

## Data-flow discipline

Think in flows, not files.

```text
source of truth → core/shared → server → transport → client → UI
```

When changing a data shape — a field, payload, or coordinate — grep the old shape across the whole codebase before coding. List the consumers, update each, check them off. Asymmetry between producer and consumer is the likely regression.

## Documentation discipline

Match existing density: bullets stay bullets, one-liners stay one-liners. Do not shout in a quiet file.

Assume the next session starts with zero context. Record gotchas as `what happened → why → how to avoid`. Update existing sections; do not duplicate.

## Documentation map

| Need | Read |
|---|---|
| Progress / backlog / shipped features | `docs/PROGRESS.md`, `docs/BACKLOG.md`, `docs/FEATURES.md` |
| Plans / specs / handoffs / investigations | `docs/planning/INDEX.md`, then linked active files |
| Non-obvious invariants / load-bearing facts | `docs/INVARIANTS.md` |
| What a domain term means | `docs/GLOSSARY.md` |
| Testing posture | `docs/TESTING.md` |
| Cross-session lessons / session audit trail | `docs/knowledge/learnings.md`, `docs/knowledge/session-history.md` |

The dev-* skills also reference per-project domain docs (e.g. `MECHANICS.md`, `UI_LAYOUT.md`, `MANIFEST/`). Create those in a consuming project as the domain requires; they are not part of this base.

## Knowledge base

Durable cross-session memory about how the work goes — session audit trail + generalized learnings. `dev-session-wrap` step 4 writes it; format primer: `docs/knowledge/README.md`.

**Declare its location here** (this repo's declaration: in-project at `docs/knowledge/`). Options: in-project `docs/knowledge/` (default — versioned with the code), an external vault/wiki path (keeps reflections out of a public repo, aggregates across projects), or `none` (wrap step 4 closes as explicit N/A).

- `session-history.md` carries commit hashes; `docs/PROGRESS.md` deliberately does not.
- `learnings.md` entries are generalized per the litmus tests at its top — session-specific nouns deleted.
- External Obsidian-style vaults add `_index.md` (engagement landing note) and `planning.md` — `planning.md` is **human-owned; never edit it**.

## Plan/spec lifecycle

Plans/specs live under `docs/planning/`. Folder placement is authoritative.

- `plans/active/` — Draft, Ready, In Progress, Paused.
- `specs/active/` — active designs or `Awaiting Plan`.
- `plans/shipped/`, `specs/shipped/` — completed/merged; shipped plans carry `Commits: <sha>..<sha>`.
- `plans/abandoned/`, `specs/abandoned/` — add a "why, successor" note before moving.
- `investigations/` — `dev-investigate` findings (handoff into `dev-brainstorm`).
- `reviews/` — `dev-reviewer` output for specs and plans.
- `handoffs/` — append-only session notes.
- `INDEX.md` — active items + latest handoff. Read first when starting; update when files move.

Frontmatter:

```yaml
---
Date: 2026-06-07
Status: Ready
Spec: planning/specs/active/…
Commits: abc123..def456
Superseded-by: planning/plans/active/…
---
```

File names: `YYYY-MM-DD-slug.md`, kebab-case, short.

### Break procedures

**Spec done, no plan:** save in `specs/active/` with `Status: Awaiting Plan`; add to INDEX. Do not write the plan in the same run.

**Plan done, not implemented:** save in `plans/active/` with `Status: Ready`; add to INDEX. The plan must be executable by the next session: paths, build sequence, data-flow impacts, done checks.

**Mid-implementation:** write `handoffs/YYYY-MM-DD-{slug}.md`; update the plan to `Status: Paused` with a "Paused at" block; update INDEX; commit all three as `docs: handoff <date>`.

When resuming: read the latest handoff first, then the paused plan. Shipped history is for audit, not orientation.

### Backend/frontend phase separation

Plans that touch both engine and UI declare separate backend and frontend phases, each with its own verification criteria — ideally implemented in separate sessions. The integration test script bridges the two: written during the backend phase, exercised during the frontend phase.

## Session wrap-up

Invoke `dev-session-wrap`. The 5 steps (update docs, commit + push, self-reflect, update notes/knowledge base, persist any new rule to memory/`CLAUDE.md`/skills) are mandatory when a feature shipped. The skill owns the per-step Definition-of-Done; do not skip wrap when a feature shipped this session.
