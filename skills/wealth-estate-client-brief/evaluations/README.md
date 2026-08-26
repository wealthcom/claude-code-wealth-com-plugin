# Estate Client Brief — evaluations

Four scenarios covering this skill's core behavior.

## Purpose

These evals verify that the skill:

- resolves the client before any other call, and disambiguates on permitted fields only
- never invents a household-wide roles answer — declines and hands off once a document is named
- never invents a family member's name — states the record's real ceiling instead
- stays out of the other three estate skills' and the tax skills' territory

## The scenarios

| File | Fixture | Gates | Blocking |
|---|---|---|---|
| `happy-path-orientation.json` | H-A | 5, 8, 10 | no |
| `household-roles-declined.json` | H-A | GAP (no household roles tool) | **yes** |
| `family-names-declined.json` | H-A | GAP (no family-member-names tool) | **yes** |
| `should-not-trigger.json` | H-A | 9 / S-12 | **yes** |

All run against **H-A** — the same shared Brady fixture the other estate skills use.

## The three axes

**Triggering** — fires on the description's orientation phrasings and paraphrases; stays silent on document-specific, asset, intake, and tax questions.

**Functional** — every assertion in `expected_behavior`, `success_criteria`, and `must_not` passes on H-A.

**Versus baseline** — fewer tool calls and turns than the skill disabled, and zero failed calls; specifically, zero calls to any tool that doesn't exist in the live registry.

## Open items this eval set depends on

- **Elicitation reachability** — whether `find_client`'s native disambiguation round-trips in the deployed transport, or always falls through to the candidate list. `happy-path-orientation.json` asserts the candidate-list path as the reliable baseline.
- **The two candidate new tools** (household-level roles, named family members) — if either ships, `household-roles-declined.json` or `family-names-declined.json` gets rewritten to a real happy path, not deleted.

## Not yet built

There is no runner. These four files are a specification, not a suite.
