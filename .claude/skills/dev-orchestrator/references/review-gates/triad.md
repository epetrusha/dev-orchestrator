# Review gate: triad

Self-deferral / vague-confidence / evidence-gap checks. Apply to any authored artifact (spec, plan, proof report).

Read the entire artifact end-to-end. For each section, answer in your own words:

1. **Self-deferral check.** Does this section push off a failure / scope item / claim with language that means "from before this work", "out of scope" (without explicit justification of why this scope edge), "covered elsewhere", "engine-proven by prior phase", "math is unit-tested", "quick workaround", "will file in BACKLOG later", or any paraphrase of the same intent? Look for SEMANTIC intent, not literal phrases.

2. **Vague-confidence check.** Does this section claim something "works", "passes", "is correct", "looks good", "handles cases", "as appropriate", or any paraphrase — without citing the specific test output, screenshot file, observed state, or verification command that proves it?

3. **Evidence-gap check.** For each completion or correctness claim, is the supporting evidence (a) cited inline by name/path/output, (b) reproducible by the consumer from the citation alone, (c) actually about what's being claimed (not a sibling test, not engine-only when the surface under change is the request/API handler, not unit-tested when the user traverses the UI)?

For ANY yes on 1, 2, or 3: cite the offending section + quote the offending text. The review verdict status is `rejected` if any check returns a violation.

Rationale (kept from the original gate text): the same model that authored the rules wrote the forbidden patterns into a Prove report in a past retrospective. Skill text loaded at invocation is not load-bearing at write-time; the explicit re-read at the gate point IS load-bearing.
