# RULE-001: Wiremock Stub Updates Only

**Rule ID:** RULE-001
**Title:** Wiremock Stub Updates Only

## Rationale

Changes confined to Wiremock mapping and stub files carry no production logic risk. They
adjust how a mock server responds during testing, not how the application behaves in
production. Running an AI evaluation against such a pull request spends time and token cost
to confirm the absence of risk that is already evident from the file paths alone. This rule
lets the pipeline recognize that case and approve it directly.

## Condition

The rule applies only when **every** changed file in the pull request matches at least one
of the following path patterns:

- `**/mappings/**/*.json`
- `**/mappings/**/*.yaml`
- `**/__files/**`
- `**/wiremock/**`

The condition is a logical AND across the whole pull request: it is satisfied only if there
is no changed file that fails to match one of these patterns.

## Action

When the condition is satisfied, the pipeline approves the pull request and posts a comment
noting that RULE-001 was matched and that no LLM evaluation was performed. The comment makes
the short-circuit visible and auditable, so that a reader of the pull request can see why no
findings were produced.

## Exclusion

If any file outside the patterns above is modified in the same pull request, this rule does
not apply. A single non-matching file is enough to disqualify the pull request from
auto-approval, in which case the pipeline proceeds to normal AI evaluation as it would for
any other change. The rule never partially applies: it either covers the entire pull
request or it does not apply at all.
