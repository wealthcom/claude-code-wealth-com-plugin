# Resolution and handoffs

Procedure for resolving a client and for the two questions this skill can't answer directly.

## Resolving the client

`Wealth:find_client` takes either a `wid` or a name, and returns a discriminated union:

- **`resolution: "DIRECT"`** — an exact `wid` resolved to one client's own file. Its `client` object is
  the whole profile: names, DOB, citizenship, relationship, marital status, state, the special-needs/
  children/pets indicators, notes, deceased flag. This is what Step 2 of the workflow reads from — no
  second call.
- **`resolution: "SEARCH"`** — a name was searched for. Carries `matches` (best interpretation first),
  `matchCount`, and which stage found each one (`CLIENT_BOOK` or `TAX_BASELINE`). If the tool itself
  resolved a single choice via its own disambiguation (see below), the SEARCH call returns a DIRECT
  payload instead — treat that as the answer, not a new question.

**Disambiguation.** When a name resolves to more than one client, `find_client`'s real code calls
`context.requestChoice(...)` — a live, implemented mechanism, not aspirational. Whether it round-trips
in the deployed transport (versus falling through to the candidate list every time) is unconfirmed —
a runtime question, not a source-reading one. Write and test against the candidate-list fallback as
the reliable path; treat a live picker as a bonus if it works, not something to depend on.

**Zero matches.** `find_client`'s own `NO_MATCH_MESSAGE` explicitly forbids concluding the household
doesn't exist — it has already checked the client book and the tax-filer names on filed returns. Pass
that framing through rather than saying "no such client."

## Handoff 1 — a household-wide roles question

"Who's the trustee," "who's the executor," "who are the beneficiaries" (asked about the household, not
one document) has no tool behind it. `get_flowchart` carries a per-node `role` field, but it's
confirmed per-document only — it requires a `vaultId` and belongs to one document, so no
coverage-gated household rollup exists or can exist from it. Ask which document, then hand off
entirely to `wealth-estate-document-deep-dive` — never attempt a rollup by calling it once per
document and stitching the results together.

## Handoff 2 — named family members or beneficiaries

Nothing on the current surface enumerates a household's members by name. `find_client`'s profile
carries booleans (has children, marital status implying a spouse) and nothing else. A name
occasionally surfaces incidentally inside one document's own extracted terms (via
`get_executive_summary`, once a document is named) — that's `wealth-estate-document-deep-dive`'s
territory, reached only after a document is named, never something this skill goes looking for on
its own. This is a real gap for future tool-building, not something to approximate here.
