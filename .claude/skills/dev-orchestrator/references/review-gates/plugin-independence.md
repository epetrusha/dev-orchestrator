# Review gate: plugin-independence (core-vocabulary independence)

Applied to specs that introduce core vocabulary — names registered in the core that extensions consume: event types, op names, status types, registry entries, label keys, template names, selectors.

**Precondition:** this gate applies only when the project has a core/extension split — a generic core plus data-driven extensions (plugins, rulesets, providers, themes, tenants) — with the extension directory and its definition files declared in the project `CLAUDE.md` or `GLOSSARY.md`. A spec in a project with no declared split closes this gate with an explicit `No core/extension split — audit N/A`.

## Audit procedure

1. Inventory extension-specific terms by reading every extension in the declared extension directory — its manifest/config, any definition files, locale bundles, and any other file introducing named primitives. (Example shape: `plugins/<id>/config.json`, `plugins/<id>/statuses.json`, `i18n/plugins/<id>/<lang>.json` — substitute the project's declared layout.)

2. For each proposed core name in the spec: if it matches or references a term meaningful in only some extensions, it is leaking. Replace with a generic primitive + extension-supplied label/data.

3. The spec must document the audit under `## Plugin-independence audit`: one entry per proposed name → verdict (generic | leaks from `<extension-id>`) → resolution.

An explicit `No engine vocabulary added — audit N/A` closes this gate when nothing is introduced.

## Principle (binding)

The core and global templates never contain a noun, label, or short-code meaningful in only a subset of extensions. Adding a new extension must require only authoring its own config/data files — never editing the core.

## Output for the reviewer agent

For each core vocabulary item: name + verdict (generic | leaks from `<extension-id>`). If any leaks, status `rejected`. If the audit section is missing from a spec that introduced core vocabulary in a project with a declared split, status `rejected`.
