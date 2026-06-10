<!-- Language-agnostic, system-agnostic catalog of design & planning mental models. Compact form: one entry per model — essence, then where it bites. Cite as "<Model> (V1 #n)" / "(V2 #n)". -->

# Dev Mental Models — compact catalog

> Vol. 1: systems thinking, design & planning. Vol. 2: abstraction, modularity & extensibility. Apply at the whiteboard stage — before a line of code is written.

## Vol. 1 — Systems thinking

1. **Separation of Concerns** — divide the system so each section owns exactly one concern (persistence, rules, transport, rendering, auth), enforced by interfaces and explicit boundaries. Bites at the first box-and-arrow diagram and whenever a module grows multiple reasons to change — a name needing "and", a test mocking 10 dependencies.

2. **First Principles Thinking** — strip to provably real constraints and reconstruct; ask what you'd build today with zero legacy — the gap to what exists is your debt map. Bites when a design feels inherited ("we've always done it this way") or requirements changed but the architecture didn't.

3. **Inversion (Design for Failure First)** — ask "what would make this fail?" and eliminate those modes before designing the happy path: pre-mortems, TDD, circuit breakers, timeouts. Mandatory for critical paths (payments, auth, data pipelines) and post-incident redesign.

4. **Second-Order Thinking** — ask "and then what?" two levels past the first consequence (cache → invalidation → eventing → ordering constraints on future sharding). Apply hardest when a solution feels "obviously good" or carries high switching cost.

5. **Theory of Constraints** — throughput is limited by exactly one bottleneck; optimizing anything else is waste. Identify → exploit → subordinate → elevate → repeat. Bites before any performance or infra investment, and when busy teams still ship slowly.

6. **Blast Radius Minimization** — don't prevent failures, contain them: bulkheads, circuit breakers, feature flags, graceful degradation, progressive rollout. At every integration or shared resource ask: "if this fails at 2am, what's the smallest part we keep running?"

7. **CAP / Consistency-Availability Trade-off** — under partition you choose consistent-with-errors (CP) or available-but-stale (AP); decide explicitly per data domain (payments CP, feeds AP), not per system. Bites at every data store, cache, or replicated-service decision.

8. **Bounded Contexts (DDD)** — different contexts correctly model the same entity differently; one God "Order" shared across billing/fulfillment/support is the failure. Bites when drawing module boundaries and when teams use the same word for different things.

9. **Volatility-Based Decomposition** — decompose by what is likely to change, not by what the system does; encapsulate each volatility (storage tech, vendor, pricing logic) in its own vault. Bites when one requirement change keeps rippling across modules.

10. **Gall's Law** — a working complex system always evolved from a working simple one; complexity designed from scratch fails. Bites against day-one microservices proposals; argues for instrument-then-evolve, and underlies Strangler Fig (V1 #17).

11. **Conway's Law / Inverse Maneuver** — systems mirror the org's communication structure; to get a target architecture, shape the team structure first. Bites before large decompositions and when "we keep rebuilding the monolith in disguise."

12. **Leaky Abstractions** — every non-trivial abstraction leaks (ORM → query plan, TCP → packet loss); never design assuming it holds — build escape hatches and understand one level below. Bites on critical-path abstraction choices and mystery performance bugs.

13. **Feedback Loops** — the tighter the loop, the cheaper the correction (unit test: seconds; production incident: days); observability is a design input, not an afterthought. Bites when designing CI/CD, monitoring, and any self-correcting mechanism.

14. **Postel's Law** — liberal in what you accept (tolerate unknown fields), conservative in what you emit (strict canonical output) — while logging anomalies, never silently swallowing them. Bites on any public contract, parser, or API where old and new clients coexist.

15. **Cynefin** — classify the problem domain first: Complicated (knowable with expertise → invest in upfront design) vs Complex (emergent → probe with safe-to-fail experiments); best-practicing a complex problem makes it worse. Bites at project start when sizing upfront architecture.

16. **Fail Fast** — surface invalid states immediately and loudly; limping along propagates corruption and hides root causes. Validate at entry points, crash on bad config rather than defaulting silently; challenge "return null/default" patterns in review.

17. **Strangler Fig** — replace a running system incrementally behind a facade, routing traffic piece by piece; the big-bang rewrite can't validate against real traffic and violates Gall's Law. Mandatory when there is no downtime budget; applies to DBs, APIs, and model swaps alike.

18. **Volatility vs Variability** — variability is config or a conditional ("EUR not USD"); volatility forces multi-module redesign unless encapsulated ("replace the payment provider"). Conflating them yields microservice explosion or fragile monoliths. Test: if this changes, how many modules move?

19. **Map vs Territory** — diagrams, schemas, and mental models of production are abstractions; treat designs as hypotheses and continuously audit actual behavior against them. Bites in architecture review ("what is this diagram not showing?") and every data-distribution assumption.

20. **Cognitive Load / Two-Pizza Team** — ownership must fit inside a small team's head; draw boundaries where two teams need minimal knowledge of each other's internals. Bites when a team can't keep up with its own incidents, and when sizing services.

## Vol. 2 — Abstraction, modularity & extensibility

1. **Open/Closed Principle** — add behavior by adding code (a new implementation of a slot), not by modifying shipped code. Refactor toward the interface when the second variation appears — not before; speculative extension points are dead weight.

2. **Liskov Substitution** — a subtype must honor the parent's full contract: no weakened preconditions, no surprise exceptions, no changed semantics. Test: swap parent for subtype — do all tests still pass? If the contract can't be honored, use composition.

3. **Interface Segregation** — many small role interfaces over one fat one; depending on a fat interface couples you to capabilities you don't use. Bites on plugin contracts, SDK surfaces, and interfaces grown by "just one more method."

4. **DIP + IoC** — high-level modules depend on abstractions; concrete wiring lives in a composition root. The foundation of hexagonal/clean architecture and all isolation testing. Apply at every boundary that might change: storage, messaging, external APIs, clocks.

5. **Hexagonal Architecture (Ports & Adapters)** — the domain core knows nothing of frameworks or infrastructure; it defines ports, adapters implement them (HTTP, DB, CLI, queue). Apply from day one when the domain must outlive infrastructure churn.

6. **Anti-Corruption Layer** — an explicit translation boundary so an external system's model never leaks into yours; the single place that absorbs third-party changes. Mandatory at legacy integrations and during strangler migrations.

7. **CQRS** — separate the write model (intent, invariants, consistency) from read models (denormalized projections per view). Apply when read and write needs genuinely diverge — scaling, shape, audit/replay; skip for plain CRUD.

8. **Event Sourcing** — the append-only event log is the system of record; state is a replayed view. Buys audit trail, temporal queries, replay, event-driven integration — at real operational cost. For domains where history is as valuable as state; skip for simple CRUD.

9. **Acyclic Dependencies** — the component graph must be a DAG; a cycle means nothing in it builds, deploys, or reasons independently. Break cycles by extracting the shared piece or inverting an edge with an interface; check at code and service level both.

10. **Stable Dependencies + Stable Abstractions** — dependency arrows point toward stability, and the more depended-upon a component is, the more abstract it should be. UI/adapters: unstable and concrete; domain/core: stable and abstract.

11. **DRY / KISS / YAGNI** — one representation per piece of knowledge (knowledge, not similar-looking code); as simple as the problem allows; build only what's needed. **Scope calibration (binding):** YAGNI governs *your own* speculation — it does **not** override a need the scope-owner has stated. "Needed" is defined by whoever owns the scope, not by what you can currently see a use for; a declared roadmap of known-required capabilities is not speculative generality. The litmus is inversion: do you actually *know* you won't need it? YAGNI is for the known-unneeded, not the merely not-yet-visible. Invoking it to defer or challenge a stated need is misapplied minimalism — widen, scope it with the user, and let them make the call.

12. **Program to an Interface** — every dependency is a contract; concrete types are named only at the composition root. The single principle that makes isolated unit testing possible; violated wherever business logic constructs its own infrastructure.

13. **CCP + CRP** — group what changes together for the same reasons (pushes components larger); don't force consumers to depend on what they don't use (pushes smaller). Calibrate granularity between the two by deployment model and system maturity.

14. **Plugin Architecture** — OCP at system level: a fixed core with declared extension points, a discovery mechanism, a lifecycle contract, and an isolation boundary. Apply when extensions come from code you don't control, or when a strategy pattern outgrows a handful of variants.

15. **Idempotency** — every networked or queued mutation must be retry-safe: an idempotency key recorded atomically with the effect; reframe actions as "apply intent-{id} exactly once." Mandatory for payments, webhooks, queue consumers, and pipeline sinks.

16. **Information Hiding (Parnas)** — each module hides one design decision likely to change; the interface promises behavior, the implementation stays swappable. The first question of any decomposition: "which decisions might we need to change later?" Each answer is a boundary candidate.

17. **Backward Compatibility / Semver** — additive changes are safe (MINOR); removals, renames, and semantic shifts are breaking (MAJOR) and need version gates and deprecation paths. What lets producer and consumer deploy on independent schedules.

18. **Composition over Inheritance** — inherit only for genuine is-a where LSP holds; otherwise hold a component and delegate. Hierarchies more than two levels deep are usually composition opportunities in disguise.

19. **Dependency Rule (Layered Direction)** — source dependencies point inward only: frameworks → adapters → use cases → entities; the domain never imports a driver. Enforce with automated architecture tests from day one.

20. **Connascence** — a taxonomy of coupling strength (name < type < meaning < position < algorithm < execution < timing < value < identity); minimize strength at boundaries, confine strong forms to the smallest scope. Use when "this feels too coupled" needs articulating precisely.

## Quick reference — Vol. 1

| # | Model | Core Question It Answers |
|---|-------|--------------------------|
| 1 | Separation of Concerns | What is the cleanest way to divide this system's responsibilities? |
| 2 | First Principles | What would we build if we started from zero today? |
| 3 | Inversion | What would make this fail, and how do we prevent it? |
| 4 | Second-Order Thinking | What are the downstream consequences of this decision? |
| 5 | Theory of Constraints | What is the one bottleneck limiting the whole system? |
| 6 | Blast Radius Minimization | When this fails, how much of the system stays up? |
| 7 | CAP Theorem | For this data store: consistency or availability under partition? |
| 8 | Bounded Contexts (DDD) | Where does each domain model begin and end? |
| 9 | Volatility-Based Decomposition | What encapsulates change so it doesn't ripple everywhere? |
| 10 | Gall's Law | Are we starting simple enough to evolve to complexity? |
| 11 | Conway's Law | Does our team structure produce the architecture we want? |
| 12 | Leaky Abstractions | What happens when this abstraction breaks down? |
| 13 | Feedback Loops | How quickly does the system learn when something is wrong? |
| 14 | Postel's Law | How do we balance input tolerance with output strictness? |
| 15 | Cynefin | Is this a Complicated problem (solvable) or a Complex one (emergent)? |
| 16 | Fail Fast | Where should the system crash loudly vs. degrade silently? |
| 17 | Strangler Fig | How do we replace the legacy system without stopping the world? |
| 18 | Volatility vs. Variability | Is this a service boundary or just a config parameter? |
| 19 | Map vs. Territory | What is our design diagram getting wrong about production reality? |
| 20 | Cognitive Load / Two-Pizza | Can one small team fully understand and own this service? |

## Quick reference — Vol. 2

| # | Model | Core Design Question |
|---|-------|----------------------|
| 1 | Open/Closed Principle | Can I add this behavior without modifying existing code? |
| 2 | Liskov Substitution | Can every implementation truly replace the abstraction? |
| 3 | Interface Segregation | Am I forcing consumers to depend on things they don't use? |
| 4 | DIP + IoC | Do my high-level modules depend on abstractions, not concretions? |
| 5 | Hexagonal Architecture | Is my domain core completely isolated from infrastructure? |
| 6 | Anti-Corruption Layer | Am I preventing the external model from polluting my domain? |
| 7 | CQRS | Should read and write concerns use separate models? |
| 8 | Event Sourcing | Should the event history be my system of record? |
| 9 | Acyclic Dependencies (ADP) | Are there any cycles in my component dependency graph? |
| 10 | SDP + SAP | Do stable components depend on more stable, more abstract ones? |
| 11 | DRY / KISS / YAGNI | Am I building only what's needed, once, simply? |
| 12 | Program to Interface | Does every dependency reference a contract, not a concrete type? |
| 13 | CCP + CRP | Are components the right size — neither too coarse nor too fine? |
| 14 | Plugin Architecture | Can behavior be extended without touching the core? |
| 15 | Idempotency | Is every mutation safe to retry without harmful duplication? |
| 16 | Information Hiding | Does each module hide a design decision that might change? |
| 17 | Backward Compatibility | Am I making only additive changes, and versioning breaks explicitly? |
| 18 | Composition Over Inheritance | Am I assembling behavior, or inheriting it unsafely? |
| 19 | Dependency Rule | Do all dependencies point inward, never outward? |
| 20 | Connascence | What is the exact type and strength of coupling I'm introducing? |
