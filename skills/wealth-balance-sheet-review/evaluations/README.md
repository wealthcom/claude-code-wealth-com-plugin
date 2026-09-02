# Balance Sheet Review — evaluations

Three scenarios. Not yet run against a harness — see "Not yet built" below.

## Purpose

These evals verify that the skill:

- loads on asset and net-worth questions — "what do the Bradys own", "where are they
  concentrated", "what's this household worth", "what are the beneficiary designations" — rather
  than routing them elsewhere
- states the ownership state (matched vs. unmatched free text) in the same breath as any figure,
  and never implies a reconciliation the data does not support
- never prints an asset identifier for a holding whose labels are incomplete
- never totals a headline estate value unless inclusion in the taxable estate is confirmed
- declines at-death modelling, scenarios, and estate-plan reconciliation rather than answering them

## The scenarios

| File | Fixture | Blocking |
|---|---|---|
| `happy-path-household-assets.json` | Brady (H-A) | no |
| `at-death-modelling-declined.json` | Brady (H-A) | yes |
| `should-not-trigger.json` | Brady (H-A) | yes |

## Schema

Same shape as the estate skills' evaluations (`expected_behavior` / `success_criteria` /
`must_not`). `fixture` is the Brady household (H-A) throughout — the primary fixture for this skill,
the hard case where assets are present but nothing is reconciled.

## Open items this eval set depends on

- **Tool confirmation** — `find_client`, `get_balance_sheet`, and `get_estate_risks` are the only
  tools this skill uses. If any are renamed, these evals need re-reconciling.
- **Ownership-state fixture** — the Brady book is authored as the unmatched (free-text ownership)
  case. The success criteria that turn on the ownership caveat assume that fixture state; a
  matched-ownership fixture would need its own happy-path eval.
- **Figures are illustrative** — any specific dollar figure referenced from the example transcript
  is a placeholder, not verified against a real `get_balance_sheet` call for a Brady-equivalent
  test client.

## Not yet built

There is no runner. These files are a specification, not a suite, matching the same state as the
estate and tax skills' evaluations.
