---
name: wealth-tax-scenarios
description: Build and model a tax position for a year the household has not filed. It starts two ways — from the advisor's own assumptions, optionally sharpened by a document dropped into the conversation, or from a filed return already on file that rolls forward as the baseline. Use whenever an advisor wants a number for a year that isn't yet filed, or is working from partial information and needs to know what's still missing — "show me 2026 for the Smiths with the new tax law updates", "what would they owe if John converts 200k to a Roth IRA", "here's what I got from my meeting, what are we looking at", "they sold a rental in the spring, how does that affect liability this year", "I don't have their return yet but...". It reports what's still missing before calculating, and asks rather than assumes. Do NOT use to read a year the household already filed, or compare one filed year with another — that's wealth-tax-executive-summary.
---

# Tax Scenarios

Models a tax position for a year that hasn't been filed. Two starting points, never mixed up:

- **Path A — from assumptions.** Nothing filed yet to work from; build the year figure by figure, optionally sharpened by a document dropped into the conversation (a paystub, a partial return draft).
- **Path B — roll a filed year forward.** Start from a return already on file, apply changes on top, price it under a target year's law.

## Critical

Read this before anything else.

- **Nothing here is saved — yet.** A modeled scenario needs a confirmation step before anything reaches a client's profile, but the live registry has no scenario-save tool today. Until one ships, if an advisor asks to "save," "lock in," or "keep" a scenario, say plainly that isn't available yet — don't imply it happened.
- **The engine calculates, you do not.** Every number comes from `run_tax_calculation` or `model_tax_scenario` — never an estimate, an approximation, or "roughly." A decline is reported as a decline, not softened.
- **Ask before assuming, every time.** Filing status, ages, state, dependents — asked, never inferred from how the advisor talks about the household. Use `get_tax_calculation_inputs` to see what's still needed and ask for exactly that.
- **When the required inputs aren't satisfied, deliver the gap list and stop.** A session that ends "here's what I still need" with no figure at all is a finished, complete answer — not a failure to push through.
- **A template is a convenience, not a boundary.** There's no catalog of named strategies. What can be modeled is bounded by what `get_tax_calculation_inputs` describes as acceptable input — for a free-form calculation exactly as for a templated one. What's never permitted is naming a strategy from memory that no calculator input expresses.
- **A change amount is a delta, not a new total.** `changeBy` on `model_tax_scenario` says how much to move a figure by — negative to reduce. Getting this backwards silently produces a scenario for the wrong household.
- **Two years, and they're different things, for `model_tax_scenario`.** `baselineYear` is the filed year the figures come from; `targetYear` is the year whose tax law prices them. Ask which target year explicitly — never assume it's the same as the baseline year, and never price a pre-2025 filed year under its own law (not supported).
- **One-time items in the baseline year don't roll forward silently.** Ask explicitly whether anything in the filed year won't repeat, and net it out with a change. Rolling it forward without asking is the characteristic failure of Path B — the filed year says it happened, and nothing in the data says it won't happen again.
- **A component the calculator has no field for is a gap, not a rounding error.** If part of what the advisor describes doesn't map to any input the calculator takes (see `reference/path-selection-and-gap-rules.md` for the known example), say plainly this can't be priced here rather than folding it into the nearest input as if it were fully accounted for.
- **Never a bare rate.** Same rule as `wealth-tax-executive-summary`: every percentage carries its dollars and income basis. No blended federal+state marginal or effective rate — keep them separate; a combined figure is dollars only.
- **This skill models unfiled years. It doesn't read filed ones.** "What did they file for 2024?" is `wealth-tax-executive-summary`'s question — hand off rather than model it.

## Quick Start

1. **Step 0 — figure out which path.** Did the advisor give a filed year to roll forward, or raw assumptions/a document? If unclear, ask.
2. **Resolve the client if one's named** — `find_client`. A pure hypothetical with no client ("a couple making about $400k...") skips this; proceed without a `wid`.
3. **Path A**: `get_tax_calculation_inputs` first, to see the full input list and what's already known; gather the rest from the advisor and any dropped-in document; call again as figures arrive; `run_tax_calculation` only once the required set is satisfied.
4. **Path B**: identify the filed baseline year and the target year; if netting out a one-time item, read the underlying figure first (`get_lookback_report` or `get_extraction_form`); then `model_tax_scenario`.
5. **Calculate, then optionally Model the change and Promote** — see "Both paths" below for what each of those means once a starting position exists.
6. **Say plainly that nothing is saved**, if the advisor's next question suggests they think it might be.

## Workflow

### Step 0 — which path

Ask directly if it isn't obvious: "should I build this from scratch, or start from their filed return and adjust it?" Don't guess. See `reference/path-selection-and-gap-rules.md` for the signals that usually make this clear without asking.

### Path A — starting from assumptions

**A1 — see what's needed.** `get_tax_calculation_inputs(taxYear, filingStatus, [values so far])`. First call with no values shows the full input list; note what's already known from the conversation.

**A2 — gather the rest.** Ask the advisor directly for what's missing. If a document is dropped into the conversation (a paystub, a K-1, a partial return), it sharpens these values — read it and fold what it shows into `values` tagged `source: "document"`.

**A3 — iterate, then calculate.** Call `get_tax_calculation_inputs` again as each answer arrives. Only call `run_tax_calculation(taxYear, filingStatus, values, ...)` once nothing required is still missing. Never skip to it with assumed values for what wasn't given.

### Path B — starting from a filed return

**B1 — identify both years.** `baselineYear` is the filed year the figures come from; ask for `targetYear` explicitly — never assume it's the same year.

**B2 — read the baseline if you need its exact figures.** Netting out a one-time item (a gain, a bonus) requires knowing its precise dollar amount — pull it from `get_lookback_report` or `get_extraction_form` before building the change.

**B3 — model it.** `model_tax_scenario(wid, baselineYear, targetYear, changes)`. `changes` is a list of `{input, changeBy}` — a delta, never a new total. An empty `changes` list is legitimate: it prices the filed year rolled forward untouched, often the first thing an advisor wants to see.

### Both paths — once a starting position exists

**Calculate** — report the number in plain language, every rate paired with dollars and income basis, attributed to the assumptions or the filed return used. For Path B, report the tax both ways (baseline vs. with changes) and the difference — the comparison is usually the actual answer the advisor wants.

**Model the change** — when the advisor layers another change onto a scenario already built this conversation ("now layer a Roth conversion on top of that"), report the new figure **as a delta against the prior result**, not a bare new number — "$X more than the 2026 baseline we just built," not just "$X."

**Promote** — nothing reaches the client's profile until the advisor confirms this specific scenario, by name, for this specific client. Batched confirmation is not confirmation. In practice today: there is no tool that performs this step yet. Until one ships, if an advisor confirms they want to keep a scenario, say plainly that nothing persists yet rather than acting as though the confirmation completed something.

### Checking withholding/estimated-payment pace

```
- Build or reuse the year's projected tax (Path A or B)
- Compare against withholding + estimated payments paid so far this year
- State whether they're on pace for safe harbor (same threshold logic as
  wealth-tax-executive-summary's Payments and safe harbor section) or will owe at filing, in dollars
```

## Constraints you must not break

1. No bare rate — dollars and income basis, every time.
2. No blended federal+state rate — dollars-only for the combined figure.
3. Required inputs missing → gap list, stop. Never a placeholder calculation.
4. A one-time item in a filed year → ask before rolling it forward.
5. A change amount is a delta, never a new total.
6. No strategy named from memory — only what a calculator input actually expresses.
7. No claim that a scenario was saved, committed, or promoted — nothing implements that today; this constraint holds until it does.
8. No fabricated figure for a component the calculator has no input for (name the gap instead) — unrecaptured §1250 gain is not supported by either calculator today.

## Answer shapes

**Path A, complete**: the figure, rate-paired, attributed to the assumptions used → one offer
**Path A, incomplete**: the gap list, as questions, and nothing else — this is a complete answer
**Path B, rolled forward**: baseline tax vs. changed tax vs. the difference → one offer
**Layered change**: the new figure as a delta against the prior scenario, not a bare number
**Unsupported component** (e.g. depreciation recapture): what's missing said plainly, what the calculator can't price named as such, no fabricated figure for that portion
**"Save this scenario"**: say plainly that isn't available yet — nothing here persists

## Common Issues

**Advisor names a "strategy"** ("what if we do a backdoor Roth"): translate it to the specific input the calculator takes; if there's no matching input, say so rather than approximating.
**A rental or property sale with depreciation involved**: ask for net sale price/basis or total gain, accumulated depreciation, holding period, suspended passive losses, and state of the property vs. residence — if the depreciation-recapture portion still has no home in the calculator's inputs once everything's gathered, say so and stop rather than lump it into the capital-gain figure.
**"What did they file last year?"**: that's `wealth-tax-executive-summary` — hand off rather than pulling a lookback report here to answer it directly.
**Advisor gives a hedge** ("about 400k"): mark it `approximate: true` on the input — that's a claim about precision, separate from where the figure came from.
**No client named, pure hypothetical**: proceed without `find_client` — nothing here requires a client on file.
**"Can we keep this for next time?"**: no promotion mechanism exists yet — say plainly it isn't available and don't pretend it saved.
**"Can you handle the depreciation recapture on that sale?"**: no — this isn't supported by either calculator today. Price what you can (the ordinary long-term gain portion) and say plainly the recapture portion isn't supported.

## Examples

- [examples/scenario-modeling-session.md](examples/scenario-modeling-session.md) — the Riley household across all three flows: rolling 2025 forward and dropping a one-time gain, layering a Roth conversion as a delta, a rental sale that hits a real calculator gap and gets a stop, and a paystub-sharpened 2026 estimate with a withholding-pace check.

## Reference

- [reference/path-selection-and-gap-rules.md](reference/path-selection-and-gap-rules.md) — how to choose Path A vs. B, the full gap-list discipline, and the known depreciation-recapture gap in detail.
