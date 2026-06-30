# Dismissal Governance — Operational Plan

This document is a plan, not a built feature. It describes how finding dismissals should
work when this pipeline is operationalized inside the organization. The proof-of-concept
deliberately does **not** implement the authority model below; it is recorded here so the
decisions are settled before they are wired, and so the rules are versioned alongside the
standards they govern.

## North star: the P1 reconstruction

Every decision here serves one goal. When a pull request is later found to have caused a
Sev-1 incident, an investigator must be able to reconstruct, from durable records alone:

- which findings the pipeline raised against that PR,
- which of them were dismissed rather than fixed,
- **who** dismissed each one, in **what role**, with **what stated reason**, and **when**,
- which version of the standards was in force at the time.

If the audit trail cannot answer those questions for any merged PR, the feature has failed,
regardless of how convenient the dismissal flow is day to day. Convenience is secondary to
accountability.

## Authority model

Write access to the repository is too wide a net to dismiss a governance finding. Dismissal
authority is therefore tiered by severity and bound to named roles, not to the general
write permission:

| Finding severity | Who may dismiss | How the role is expressed |
|---|---|---|
| advisory | Tech lead(s) of the team that owns the repo | Membership in the repo's `tech-leads` team |
| required | Tech lead(s) of the team that owns the repo | Membership in the repo's `tech-leads` team |
| escalate | A full-time codeowner designated for governance calls | Membership in the org-level `fte-codeowners` team |

The distinction is deliberate. Advisory and required findings concern the team's own code
and are the tech leads' call. An `escalate` finding exists precisely because it needs
judgment beyond the immediate team — a second HTTP client, a duplicated dependency, a
cross-cutting risk — so dismissing one is reserved for the handful of full-time engineers
the organization holds accountable for that class of decision.

Roles are resolved by team membership at dismissal time, checked against the GitHub API
(`GET /orgs/{org}/teams/{slug}/memberships/{user}`) using the authenticated comment author —
not by `author_association`, which only distinguishes broad permission bands and is not
specific enough to name a tech lead. A dismissal comment from anyone outside the required
team is acknowledged and rejected, with the rejection itself recorded.

**Dependency to flag now:** this model requires a GitHub **organization** with teams. The
current POC repository is under a personal account, which has no teams, so the authority
model cannot be exercised until the repos move into the org. This is the same constraint
that blocks the `escalate` reviewer-request path, and both should be resolved together when
the pipeline moves to the org.

## Persistence across pushes

Dismissals must survive new commits. A tech lead who has reviewed and dismissed a finding
should not have to re-dismiss it every time the author pushes an unrelated change; that
tedium would push people toward blanket dismissals, which is the opposite of what we want.

This requires giving each finding a **stable identity** that is independent of the per-run
`findingId` (which is regenerated every evaluation). The proposed key is:

```
standardId + file + sha256(normalized offending snippet)
```

On each re-evaluation, after the model produces fresh findings, the pipeline re-applies any
prior dismissal whose stable key matches a current finding. The carry-forward rule is the
governance-relevant part:

> A dismissal carries forward **only while the offending content is unchanged.** If the
> code that triggered the finding is materially edited (the snippet hash changes), the
> dismissal does **not** carry — the finding is treated as new and must be dismissed again.

This prevents a dismissal from silently laundering a *changed* violation, while sparing
reviewers from re-dismissing findings that genuinely have not moved. Line numbers are
deliberately excluded from the key, because they shift for unrelated reasons; the content
hash is what matters.

The set of active dismissals (each carrying its stable key and full provenance — see below)
is persisted as machine-readable state on the PR, so that both the evaluation run and any
future re-evaluation can read it. The proof-of-concept persistence mechanism is a single
hidden block inside a sticky pipeline comment on the PR; the operational version may move
this to a more durable store, but the contract — "the current findings and their dismissal
state are re-readable between runs" — is the same.

## The audit record

The current finding schema records only `disposition` and a bare `dismissalReason` string.
That is insufficient for the P1 reconstruction: it captures *that* something was dismissed
and *why*, but not *who*, in *what role*, or *when*. The plan extends the finding with a
structured, nullable `dismissal` object, required whenever `disposition` is `dismissed`:

```json
"dismissal": {
  "dismissedBy":   "github-login",
  "dismisserRole": "tech-lead | fte-codeowner",
  "reason":        "non-trivial free text, validated non-empty",
  "timestamp":     "ISO 8601",
  "atCommitSha":   "the head SHA the dismissal was made against",
  "standardsRef":  "the pinned standards ref in force at dismissal time"
}
```

Three properties make this forensically sound:

- **Authenticated actor.** `dismissedBy` is taken from the GitHub event's comment author,
  which cannot be spoofed by content in the comment body. The role is resolved server-side
  from team membership, not claimed by the commenter.
- **Mandatory, meaningful reason.** Empty reasons, or placeholders like "n/a", are rejected.
  The reason is the human justification an investigator will read first.
- **Append-only history.** Dismissals are recorded as events, not in-place mutations. Editing
  or deleting the dismissal comment after the fact does not erase the recorded event; comment
  edits are either ignored or recorded as new events, never silently applied. The trail is
  what survives, not the comment.

At merge, the final findings — each carrying its `dismissal` provenance — are written to
`audit-log/{prNumber}-{commitSha}.json` in the application repository, exactly as the
existing audit-log design specifies. Because that file is committed to git, the record is
immutable and timestamped by the merge itself, and because every finding already carries the
`evaluatorVersion` (the pinned standards ref), the investigator also knows which standards
were in force. The audit log is the durable artifact the P1 reconstruction reads from; the
sticky comment is only the in-flight working copy.

This `dismissal` object is a proposed change to `findings-schema/finding.schema.json` and, per
this repository's rules, is itself subject to architecture review before adoption.

## The enabling mechanism: a mutable gate

A dismissal handler runs in response to a PR comment — a different event from the pull
request evaluation — so it cannot change the evaluation job's pass/fail. The merge gate must
therefore live in a dedicated commit-status context (proposed: `governance/gate`) that both
the evaluation run and the dismissal handler can set on the head commit. Branch protection
requires that status context rather than the evaluation job. The evaluation sets it; a
dismissal that clears the last blocking finding re-sets it to passing. This is an
implementation detail rather than a governance decision, but it is a prerequisite, so it is
recorded here.

## Incident replay walkthrough

Given a PR number later implicated in a P1, an investigator:

1. Opens `audit-log/{prNumber}-{commitSha}.json` in the application repository.
2. Reads every finding the pipeline raised, with its severity and standard.
3. For each `dismissed` finding, reads the `dismissal` object: who dismissed it, in what
   role, their stated reason, the timestamp, and the commit it was made against.
4. Checks out the standards repository at the finding's `evaluatorVersion` to see the exact
   text of the standard that was waived.
5. Concludes with a defensible, named account of which risks were flagged, which were
   accepted, by whom, and on what basis — without relying on anyone's memory.

That walkthrough is the acceptance test for this whole feature.

## Phasing

| Phase | Scope |
|---|---|
| Proof of concept (now) | No authority model, no role checks. Dismissal mechanics may be stubbed or omitted. This document stands in for the unbuilt governance. |
| Operational v1 | Move repos into the org; create `tech-leads` (per repo) and `fte-codeowners` (org) teams; implement the tiered authority checks, the `dismissal` schema extension, persistence with carry-forward, and the `governance/gate` status. |
| Operational v1 hardening | Append-only event ledger, comment-edit handling, and the merge-time audit-log commit wired to the sticky-comment state. |

## Open decisions

A few choices remain before operational v1 is specified completely:

- Whether an `escalate` dismissal additionally requires a second `fte-codeowner` to concur,
  rather than a single sign-off, given its blast radius.
- Whether dismissals expire — e.g., a dismissal older than N days, or made against a
  standards ref that has since changed, is re-surfaced for re-confirmation.
- Whether a dismissed-then-materially-changed finding (which loses carry-forward by the rule
  above) should notify the original dismisser, so they know their waiver lapsed.
