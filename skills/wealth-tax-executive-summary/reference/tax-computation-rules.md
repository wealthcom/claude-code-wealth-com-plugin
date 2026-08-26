# Tax computation and presentation rules

These rules exist because a filed return holds several figures that look interchangeable but
aren't, and the wrong pick produces an answer that's confidently wrong rather than obviously
incomplete.

## Effective rates

Take the effective rate **whole**, from the return's own rate summary (the same figure
`get_lookback_report` returns and the one the Tax Planning screen shows). Never:

- average two years' effective rates
- recompute one from total tax ÷ AGI (that's a different, looser number than the return's own
  effective-rate line)
- reconcile a federal and state effective rate into one blended figure

If an entry's basis can't be identified (no income figure in the return reproduces the rate),
give the dollar tax and drop the percentage. Say plainly that the rate isn't shown because it
can't be reproduced — don't silently omit it.

## The ordinary marginal bracket

Total taxable income is not the number to bracket. Preferential income — long-term capital gains
and qualified dividends — sits on top of ordinary income and is taxed at its own schedule, not the
ordinary brackets. So:

1. Take taxable income.
2. Subtract net long-term capital gains and qualified dividends.
3. Find the bracket for what's left. That's the ordinary marginal bracket.

Skipping step 2 and bracketing total taxable income is the single most common way this section goes
wrong — it usually overstates the bracket, sometimes by a full tier, because it's counting
preferentially-taxed income as if it were ordinary.

## State rate presentation

Branch on the state's structure (from `get_state_tax_constants`):

- **GRADUATED** — present an effective rate and a marginal bracket, same as federal, each with
  dollars and basis.
- **FLAT** — one rate applies to all taxable income; present it as the rate, not as a "bracket."
- **NONE** — no state income tax. Say so plainly; this is a complete answer, not a gap.
- **NARROW_BASE** — the state taxes only certain income types (e.g., interest and dividends, not
  wages). Say which income types are taxed before giving a rate — a bare rate here is misleading
  because it doesn't apply to the household's whole income picture.

## Combined federal + state figure

Give the combined tax **in dollars only** — "$X federal plus $Y state, $Z combined." Never add the
two rates into one marginal or effective percentage; federal and state brackets don't stack in a
way a single blended rate can represent honestly.

## Capital-gains bracket positioning

Position net long-term gains against that year's breakpoints from `get_federal_tax_constants`.
State the bracket name (0% / 15% / 20%) and the dollar distance to the next breakpoint, not just
"they're in the 15% bracket" — the distance is usually what the advisor actually wants (e.g., "how
much more can they realize before crossing into 20%").

## IRMAA positioning

IRMAA tiers are based on MAGI from **two years prior** to the premium year, not the filed year's
own MAGI against that year's tiers — check which year's tiers `get_federal_tax_constants` returned
and confirm it's the premium-determining year before stating a tier. State the tier the household
is in and the dollar distance to the next tier, not just the tier name.

## Safe harbor

Safe harbor is met by paying, through withholding and estimated payments combined, the lesser of
90% of the current year's tax or 100% of last year's tax (110% if last year's AGI exceeded
$150,000, or $75,000 married filing separately). State which threshold applies to this household
before saying whether they met it.
