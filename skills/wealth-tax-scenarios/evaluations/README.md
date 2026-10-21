# Tax Scenarios — evaluations

Five scenarios. Three of them gate the launch — the promotion/save honesty check, the
depreciation-recapture gap and the early-distribution owner check are all places a
plausible-sounding but wrong answer is the likely failure mode, the same class of risk
`wealth-balance-sheet-review`'s no-false-reconciliation gate protects against.

## Purpose

These evals verify that the skill:

- picks Path A vs. Path B correctly and doesn't blend them
- never runs a calculation on a missing required input — delivers the gap list and stops instead
- never rolls a one-time item forward silently
- never claims a scenario was saved — the Promote step is defined in the doc but has no named
  implementation, and none exists in the live registry either
- never prices a component (unrecaptured §1250 gain) the calculator has no input for, and says so
  rather than folding it into a nearby field
- asks whose account an IRA or pension withdrawal comes from before pricing a joint return with one
  filer on each side of 59½, and never puts a figure on the 10% additional tax
- pairs every rate with dollars and income basis, same as `wealth-tax-executive-summary`

## The scenarios

| File | Fixture | Blocking |
|---|---|---|
| `happy-path-rollforward-and-layer.json` | Riley | no |
| `depreciation-recapture-gap.json` | Riley | **yes** |
| `save-scenario-declined.json` | Riley | **yes** |
| `early-distribution-owner-unstated.json` | none | **yes** |
| `should-not-trigger.json` | Riley | yes |

`none` means an unnamed household on Path A: no client lookup and no fixture.

## Still open

- **Promotion isn't live yet.** `save-scenario-declined.json` is correct today but has a shelf
  life — once a save/promote tool ships, this eval must be rewritten to a real happy path, not
  deleted, so it doesn't silently go stale as a false negative.
- **Unrecaptured §1250 gain isn't supported by either calculator.** `depreciation-recapture-gap.json`
  asserts the current, correct fail-closed behavior for that gap.
- `list_tax_strategies` doesn't exist; no eval here assumes a named-strategy catalog.
- **The owner rule waits on the calculator.** `early-distribution-owner-unstated.json`'s asking gate
  holds either way, but the two owners price differently only once the calculator charges a joint
  position the 10% unless both filers are 59½ or older.

## Not yet built

There is no runner. These files are a specification, matching the same state as every other
skill's evaluations in this bundle.
