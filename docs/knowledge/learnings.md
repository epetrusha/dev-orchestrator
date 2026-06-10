# Learnings

Generalized lessons from development. Read first; apply throughout the session.

## How to write entries here

A learning is useful only if it applies to many DIFFERENT situations — not the same situation recurring. Litmus before writing: how many EXACTLY THE SAME situations would have to happen for this entry to help? If the answer is "few," the entry is overfit to the session that produced it — crystallize one level higher. The trap: when a correction lands, the reflex is to encode the SHAPE of what happened (the specific words, file paths, framings) as the lesson. The shape is session-specific; the actual lesson is the principle underneath, which applies to situations that look nothing like the trigger.

Portability tests: would this apply to a different feature in a different domain? Could a non-software example illustrate it? Delete every session-specific noun — does the principle still stand? Headlines are direct and declarative, no hedging — don't water a real insight into vague advice ("think about context" conveys nothing). Reject conditional cascades: "when X, but if Y, unless Z" is session-overfit dressed up as nuance; the principle stands on its own, exceptions live in judgment at application time. When in doubt: "reading this in six months in a totally different situation, would it help me directly, or would I first have to translate session-specific framing?" If it needs translation, rewrite it.

**Entry form:** `- **Declarative headline.**` then one dense paragraph: the reflex/failure mode → the discipline that counters it → a litmus test → a non-software illustration → the generalization statement.

---

- **Generated outputs are symptoms; fix the generator or source definition.** (Seed example — keep or delete.) When a feature depends on data that can be recreated, reseeded, or regenerated, editing the generated output is a temporary illusion. The durable fix belongs in the source that produces it: fixture builder, content definition, schema default, seed script, migration, or canonical config. If the regenerated state would lose the behavior, the behavior was never actually shipped. Non-software illustration: correcting a printed menu while the ordering system still has the old price means tomorrow's print run reintroduces the error. Generalizes to any workflow where runtime artifacts are downstream of authored source.
