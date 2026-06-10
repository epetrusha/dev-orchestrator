# Review gate: forbidden rationales

Self-deferral or substitution rationales that are forbidden in plan bodies, spec bodies, and proof reports. Any occurrence in main-body content is a violation.

Allowed only in an explicit §Out-of-scope section with a one-line justification stating which scope edge applies.

## Forbidden phrases (and their paraphrases)

1. "Pre-existing — predates this work / out-of-scope for this row/phase"
2. "Covered elsewhere — integration tests assert this / engine-proven by prior phase"
3. "Math is unit-tested"
4. "User will see it next session anyway"
5. "Plan said X but Y is close enough" (substitution without assent)
6. "Quick workaround so I can keep moving"
7. "Will file in BACKLOG later"
8. "Engine pipeline proves the chain — UI layer is incremental"

## Output for the reviewer agent

For each match: cite section + quote phrase + rule number (1-8). Status `rejected` if any match in main-body content.
