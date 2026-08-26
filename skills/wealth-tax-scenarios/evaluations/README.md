# Tax Scenarios — evaluations

Four scenarios. Two of them gate the launch — the promotion/save honesty check and the
depreciation-recapture gap are both places a plausible-sounding but wrong answer is the likely
failure mode, the same class of risk `wealth-balance-sheet-review`'s no-false-reconciliation gate
protects against.

## Purpose

These evals verify that the skill:

- picks Path A vs. Path B correctly and doesn't blend them
- never runs a calculation on a missing required input — delivers the gap list and stops instead
- never rolls a one-time item forward silently
- never claims a scenario was saved — the Promote step is defined in the doc but has no named
  implementation, and none exists in the live registry either
- never prices a component (unrecaptured §1250 gain) the calculator has no input for, and says so
  rather than folding it into a nearby field
- pairs every rate with dollars and income basis, same as `wealth-tax-executive-summary`

## The scenarios

| File | Fixture | Blocking |
|---|---|---|
| `happy-path-rollforward-and-layer.json` | Riley | no |
| `depreciation-recapture-gap.json` | Riley | **yes** |
| `save-scenario-declined.json` | Riley | **yes** |
| `should-not-trigger.json` | Riley | yes |

## Still open

- **Promotion isn't live yet.** `save-scenario-declined.json` is correct today but has a shelf
  life — once a save/promote tool ships, this eval must be rewritten to a real happy path, not
  deleted, so it doesn't silently go stale as a false negative.
- **Unrecaptured §1250 gain isn't supported by either calculator.** `depreciation-recapture-gap.json`
  asserts the current, correct fail-closed behavior for that gap.
- `list_tax_strategies` doesn't exist; no eval here assumes a named-strategy catalog.

## Not yet built

There is no runner. These files are a specification, matching the same state as every other
skill's evaluations in this bundle.
