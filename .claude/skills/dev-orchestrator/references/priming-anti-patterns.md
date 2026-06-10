# Priming anti-patterns

Vocabulary bank consumed by the orchestrator's session-entry priming gate and the brainstorm skill's brainstorm-entry priming gate. Each entry has a `phase-tag`; the consuming gate selects entries whose phase-tag matches its phase.

Each entry format:

```
## <name>
**Phase:** <session-entry | brainstorm-entry | brainstorm-iteration | fix-iteration>
**Signal:** <concrete observable in my own draft thinking or in the conversation>
**Countermove:** <what to do instead>
```

---

## Anchoring on input wording
**Phase:** session-entry
**Signal:** My framing mirrors the wording in the user's prompt or in an observation/handoff doc without examining whether that framing is incomplete.
**Countermove:** Paraphrase the request in my own words. Identify what the user might not have considered.

## Familiarity bias
**Phase:** session-entry
**Signal:** My decomposition produces a list of similar items (same category as what I just read).
**Countermove:** Ask "what category of mechanism would actually solve this?" Check whether a structurally different option exists.

## Catalog not applied
**Phase:** session-entry
**Signal:** I know a reference (mental models, gates, observations) exists but am about to draft without applying it.
**Countermove:** Invoke the relevant PRODUCE step before drafting.

## Cheap-by-default
**Phase:** session-entry
**Signal:** I'm picking the smaller or lighter option without testing whether it actually solves the failure mode under address.
**Countermove:** State the failure mode. Check whether the cheaper option closes it by construction, or only by honor.

## Mirroring user proposals as approval
**Phase:** brainstorm-iteration
**Signal:** Response leads with "confirmed:" or rolls forward user's design without independent evaluation.
**Countermove:** Examine the proposal on its merits. List pros + cons + my opinion before adopting.

## Performative agreement
**Phase:** brainstorm-iteration
**Signal:** I say "great point" or "that's a clever consolidation" without actually evaluating.
**Countermove:** State what makes it good or bad. If I don't have an opinion, say so.

## Wall of text
**Phase:** brainstorm-iteration | fix-iteration
**Signal:** Draft > 30 lines for a non-spec response.
**Countermove:** Cut to the decision-relevant signal only.

## Batched questions
**Phase:** brainstorm-iteration | fix-iteration
**Signal:** More than one `?` in draft.
**Countermove:** Pick the one decision the user needs to make now. Defer the rest.

## Code-identifier jargon without context
**Phase:** brainstorm-iteration | fix-iteration
**Signal:** Backticked identifier (function, API event, property, agent name) introduced without a plain-language explanation alongside.
**Countermove:** Explain what it does in ≤ 5 words before naming it.

## No context preamble
**Phase:** brainstorm-iteration | fix-iteration
**Signal:** Response leads with the answer without one-line framing of what's being answered.
**Countermove:** One-line context first, then the answer.

## Reality-check skipped
**Phase:** brainstorm-entry | brainstorm-iteration | fix-iteration
**Signal:** I'm about to claim what existing engine / codebase / external source does or contains without grep / Read / WebFetch / WebSearch evidence. Memory-shaped phrases in draft: "the engine already supports", "X exists today", "the test suite has Y helper", "the third-party API does Z", "the spec canonically says W".
**Countermove:** Stop drafting. Grep / Read the codebase for engine + test-suite claims (`file:line` citation); WebFetch / WebSearch canonical sources for external-system claims (URL citation). Add the citation inline in the spec/plan body. The reviewer agent rejects unverified engine-state claims and API hallucinations on every pass — pay the cost up front.
