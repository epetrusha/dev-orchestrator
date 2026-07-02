# Design Principles

> The named set of *this project's* generative design rules — the ones applied when making design decisions. Distinct from ADRs (a single past *decision* + its why), from [`INVARIANTS.md`](INVARIANTS.md) (*facts* the code must hold), and from behavior docs. A register, not a second source of truth: each leaf names a rule and points to its home (an ADR, a reference, a learning), never restating it.
>
> `dev-deliberator` reads this file as its `north-star-path` — the bar every design answers to and the tests it breaks shapes with. Ships as an adaptable seed: the five axioms are general and usable as-is; the **North Star is project-specific — write yours.** A project that keeps no principles doc points `north-star-path` at [`INVARIANTS.md`](INVARIANTS.md) + the ADR index instead.

---

## North Star — what every principle serves

The one thing this system is *for*, in a few lines: the mission every design decision serves and the long-term intents it commits to. An axiom earns its place only by serving it. Derive two or three design tests from it that every change must answer.

---

## The five axioms

### 1. Right level, right place, right shape

Is the abstraction at the correct altitude, living where it belongs exactly once, shaped cleanly?

- **Decompose by volatility, not similarity** — split by what changes together, not by surface resemblance; a block that fuses validation, mutation, and effects is an extraction candidate its callers can reuse.
- **Weigh cost by architectural impact, not line count** — a wide but mechanical edit that provides leverage or unlocks functionality is cheap; a few lines that introduce a coupling or force a migration are not.

### 2. Reuse, don't repeat

One home per piece of knowledge; don't write what already exists.

- **Read before writing** — before a new test, util, or integration, read the existing code that solves the same class of problem, and extend it.
- **Find the second call site** — the logic you're extracting is usually already re-implemented elsewhere (a validation and its preview; two callers of one rule). Consolidation pays off across both.
- **Derive from one declaration** — vocabularies, validators, and their tests derive from a single source; a second hand-maintained list is drift waiting to happen.
- **The shared core stays pure** — it carries no dependency on any one consumer, transport, or framework.

### 3. Don't build on shaky ground

Build on sound foundations; when the ground is shaky, fix it rather than route around it.

- **Fix faulty abstractions, don't anchor** — when the code you build on is a hack, fallback, or leaky abstraction, redesign it; don't preserve, wrap, or rename it. The smallest change against a wrong structure is not the smallest *correct* change.
- **Test through real use** — prove end-to-end against the real use case, not by mocking your own internals; a mock encodes an assumption that drifts from reality.
- **User data is sacred** — internals refactor freely, but a persisted or published shape changes only behind a migration.

### 4. Generic core, cases at the edges

The core encodes the *shape* of a mechanism; each case supplies its values and names.

- **Behavior in code, values in config** — the values that vary by case (limits, thresholds, defaults) live in data the core reads through a declared boundary; no magic-number defaults. If another case could change it, it's config.
- **No case-specific branches in the core** — a `case === 'X'` test in generic code is the violation; the special behaviors of one case are instances of a general pattern.
- **Address by role, not by name** — the core reaches a value by its role (the primary key, the active item), never one case's literal identifier. A case-specific literal in generic code is a leak.

### 5. One source of truth for state

One canonical answer to a given state question, one controlled path to change it.

- **One reader per question** — one place answers "what is X now?"; consumers read it rather than each re-deriving. Two sites computing the same predicate is the failure mode.
- **One writer per piece of state** — mutations go through a single controlled path, not scattered writes.

---

## Related

- `adr/README.md` — the per-decision rationale each leaf points to.
- [`INVARIANTS.md`](INVARIANTS.md) — the facts the code holds as a consequence of these principles.
- `CLAUDE.md` — the agent-facing index that points here.
