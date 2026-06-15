# Review gate: abstraction & reuse (right-level encoding)

Applied to specs that introduce a primitive, module, type, or named definition — anything another part of the system builds on or consumes.

## The principle

Before adding code, separate the **invariant essence** from the **per-case variation**. The essence gets one home; variation is expressed at the edges — a parameter, a config value, a swappable strategy, an adapter. This is the constructive, design-time form of three models: DRY/KISS/YAGNI (V2 #11), Information Hiding (V2 #16), Volatility vs Variability (V1 #18).

Two failure modes it rejects:

1. **Near-duplicate variant.** A new case is mostly an existing one. Forking a tweaked copy duplicates the essence — reuse the original and lift the differing part out as a parameter / config / swappable piece, so the new case is a specialization, not a second implementation.
2. **Per-consumer bespoke.** A new capability that more than one consumer needs essentially the same. Implementing it per consumer — or welding it to one — scatters the essence. Build it once as a core module; let each consumer reach it through a thin adapter/wrapper.

Unifying test: if two things share an essence, that essence lives in one place; differences sit in a thin variable layer around it.

## When the shared abstraction is required (YAGNI does not excuse skipping it)

Build the shared core + adapters now when ANY of:
- the user's stated direction or intent makes clear several consumers will exist;
- a few seconds' reasoning makes other consumers obvious;
- grounding — the code/docs read this session — shows the other consumers already exist.

Otherwise — a merely hypothetical future consumer — YAGNI applies: do not pre-build the abstraction. (YAGNI governs speculation, never needs that are stated, evident, or grounded.)

## Audit procedure

For each primitive/module/type/name the spec introduces:
1. **Duplication check.** Does something equivalent already exist? A near-variant resolves to reuse + parameterization, not a copy.
2. **Consumer check.** Does this capability have — or, by the triggers above, evidently will have — more than one consumer? If so, the essence must live in one shared home with per-consumer adapters, not fused to one caller or duplicated.
3. **Leak check.** Does any shared/core definition encode a detail meaningful to only one case? Push that detail into the variation layer.

Document under `## Abstraction & reuse audit`: one entry per introduced name → verdict (right-level | duplicates `<existing>` | fuses case `<X>`) → resolution. Nothing introduced → `No shared primitives introduced — audit N/A`.

## Special case: plugin / extension architectures

When the consumers are data-driven extensions (plugins, providers, themes, tenants, rulesets) declared in `CLAUDE.md`/`GLOSSARY.md`, the consumer check takes a concrete form: the core must not name a term meaningful in only some extensions, and adding an extension must require only authoring its own config/data — never editing the core. Inventory the declared extension directory; for each proposed core name, if it references a term meaningful in only some extensions, it leaks → replace with a generic primitive + extension-supplied data.

## Principle (binding)

A shared definition encodes the essence common to all its cases and nothing specific to one of them. Adding a new case — a variant, a consumer, or an extension — extends via parameter/config/adapter/data, never by duplicating the essence or editing the core to know about the new case.

## Output for the reviewer agent

For each introduced name: name + verdict (right-level | duplicates `<existing>` | fuses case `<X>` | leaks from `<extension-id>`). Any duplicates/fuses/leaks → status `rejected`. Missing audit section in a spec that introduced shared primitives → status `rejected`.
