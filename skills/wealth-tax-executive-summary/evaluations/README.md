# Tax Executive Summary — evaluations

Three scenarios. Not yet run against a harness — see "Not yet built" below.

## Purpose

These evals verify that the skill:

- loads on filed-return questions phrased as "executive summary," "tax overview," "summarize
  their return," or "what's their tax situation" — even without the words "executive summary"
- stays descriptive: never models a change, never recommends a strategy
- never presents a bare rate — every percentage carries its dollars and income basis
- routes estate documents and forward-looking ("what if") questions elsewhere rather than
  answering them itself

## The scenarios

| File | Fixture | Blocking |
|---|---|---|
| `happy-path-filed-return.json` | Riley | no |
| `no-filed-return-declined.json` | Riley | yes |
| `should-not-trigger.json` | Riley | yes |

## Schema

Same shape as the estate skills' evaluations (`expected_behavior` / `success_criteria` /
`must_not`), adapted for tax: `fixture` is the Riley household throughout (distinct from the estate
skills' Brady fixture).

## Open items this eval set depends on

- **Tool confirmation** — `find_client`, `get_lookback_report`, `get_extraction_form`,
  `get_federal_tax_constants`, `get_state_tax_constants` are all confirmed live in the current
  registry. If any are renamed again, these evals need re-reconciling.
- **IRMAA/safe-harbor arithmetic** — the worked figures in `reference/tax-computation-rules.md` and
  the example transcript are illustrative, not verified against the real tax engine's output for
  this fixture. Treat any specific dollar figure in an eval as a placeholder until it's checked
  against a real `get_lookback_report` call for a Riley-equivalent test client.

## Not yet built

There is no runner. These files are a specification, not a suite, matching the same state as the
estate skills' evaluations.
