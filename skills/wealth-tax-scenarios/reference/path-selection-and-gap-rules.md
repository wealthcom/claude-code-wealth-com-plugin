# Path selection and the gap-list discipline

## Choosing Path A vs. Path B

| Signal | Path |
|---|---|
| Advisor gives raw figures or a "what if a couple made about $X" hypothetical | A |
| Advisor drops a document into the conversation (paystub, partial return) with no filed year named | A, sharpened by the document |
| Advisor says "roll them forward," "start from last year," or names a filed year as the starting point | B |
| Advisor asks to layer a change onto a scenario already built this conversation | Continue on whichever path was already in use |

If it's genuinely ambiguous, ask: "should I build this from scratch, or start from their filed
return and adjust it?" — don't guess.

## The gap-list discipline

`get_tax_calculation_inputs` answers two different questions, and both matter:

1. **Can this run at all?** Some inputs are required (tax year, filing status) — without them
   there's nothing to calculate.
2. **How complete is the picture?** Beyond the required minimum, more inputs mean a more accurate
   number. A calculation can "run" on thin information and still be worth flagging as thin.

When required inputs are missing, stop and ask — don't run the calculation on a placeholder. When
optional-but-material inputs are missing (e.g. no capital gains figure for a household that
mentioned selling stock), say the number is based on what's known and name what's not.

A gap list is a complete answer. Don't apologize for it or treat it as a failure to push past —
"here's what I still need before I can give you a number" is exactly what this skill should say
when that's true.

## The known calculator gap: unrecaptured §1250 gain

`run_tax_calculation` and `model_tax_scenario` take a fixed, enumerated list of inputs. That list
has `capital_gain_or_loss` and related capital-gains fields, but **no separate input for
unrecaptured §1250 gain** — the portion of a depreciated real property sale taxed at a 25% maximum
rate rather than ordinary capital-gains rates. This isn't supported by either calculator today, and
there's no committed plan to add it. The behavior below (price what's supported, decline what
isn't, never fabricate a number for the unsupported portion) is the correct and current behavior.

This means a rental or investment-property sale involving depreciation recapture cannot be fully
and correctly priced by either calculator today. The advisor's answer typically comes as **raw sale
figures rather than a pre-split gain**: net sale price $500,000, adjusted basis $350,000 (→ total
gain $150,000), and depreciation taken over the ownership period of $50,000. Working out that
$50,000 of the
$150,000 gain is unrecaptured §1250 (bounded by the depreciation taken) and the remaining $100,000
is ordinary long-term capital gain is now this skill's job, not something the advisor hands you
pre-computed — so:

- Gather the raw figures: net sale price and adjusted basis (or total gain directly), accumulated
  depreciation, holding period, suspended passive losses, this year's rental income, and state of
  the property vs. residence.
- Derive the split yourself: unrecaptured §1250 gain is the lesser of the depreciation taken or the
  total gain; the remainder is ordinary long-term capital gain.
- Price the long-term-capital-gain remainder normally through the calculator.
- Say plainly that the unrecaptured §1250 portion isn't broken out by this calculator, so the
  number you can give covers the capital-gains portion only — not the full sale.
- Don't fold the recapture amount into `capital_gain_or_loss` as if it were taxed the same way;
  that would understate the actual liability without saying so.
- This is a product gap, not an advisor-information gap — don't attempt to paper over it with
  better prompting.

## Named "strategies" have no catalog

Advisors describe strategies by name (e.g., "a Roth conversion," "a QCD"). There is no tool that
lists strategy identifiers: "a template is a convenience, not a boundary." Translate directly:

| Advisor language | Calculator input |
|---|---|
| "Convert $200k to a Roth" | `roth_conversion_taxable`, changeBy/amount 200000 |
| "They're putting more into their 401(k)" | `taxpayer_401k_contribution` / `spouse_401k_contribution` |
| "They sold a rental" | `rental_royalty_income`, `capital_gain_or_loss`, or the Schedule E inputs, depending on what's actually being asked — see the recapture gap above |
| "A bigger charitable gift" | `charitable_cash` or `charitable_noncash` |

If the advisor's language doesn't map cleanly to one of the calculator's enumerated inputs, say so
rather than picking the closest-sounding one and hoping.
