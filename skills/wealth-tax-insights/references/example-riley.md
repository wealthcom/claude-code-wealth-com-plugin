# Tax Insights — Riley Family — Tax Year 2025

> Reference output. Match this for section order, tone, and level of detail.
>
> COPY THE SHAPE, NEVER THE NUMBERS. The Rileys are fictional and every figure below belongs to
> them alone. Take every figure in a real review from that client's own return.
>
> Same household as the executive summary's reference example, deliberately. Read the two side by
> side: the summary states the Rileys are in the top IRMAA tier and carry a 35% marginal bracket.
> Nothing below restates either.
>
> Note that no `## Anomalies` section appears — the reconciliation and consistency checks all
> tied. That is the common case, and an empty section is omitted rather than padded. The
> near-misses are recorded under "Checked, nothing observed," which is where their value is.

## Gaps & Follow-Up

**Is there a reason no SEP or solo 401(k) contribution appears against the Schedule C income?**
The return shows $34,000 of net profit from the sole proprietorship and no retirement plan
deduction on Schedule 1. Both David and Victoria's wage income is separate from this business.

**Is either filer covered by a high-deductible health plan?** No HSA contribution appears, and
neither W-2 carries a Box 12 code W for employer HSA contributions. David is 65 and Medicare
enrollment forecloses an HSA for him entirely, but Victoria is 63, and whether she has HDHP
coverage outside her employer's reporting is not visible on the return.

**Is there self-employed health insurance being paid outside the return?** No self-employed
health insurance deduction appears on Schedule 1 despite the Schedule C activity. Employer
coverage through the W-2 employment would explain it.

**Does the Schedule C business use a home office, a vehicle, or a cellphone?** Gross receipts of
$48,000 carry $4,200 of cost of goods sold and $9,800 of expenses. Home office, auto, and
cellphone do not appear among them.

**Is depreciation being claimed on the rental property?** Schedule E reports $36,000 of rental
income against $27,930 of expenses, with no depreciation line visible in the extraction. Rental
income without depreciation is unusual enough to confirm rather than assume.

**Is there a second property?** The return reports roughly $45,000 of real estate taxes, against
a Schedule E showing one rental. That level of property tax points to either an additional
property or a high-value primary residence.

## Triggers

The Rileys' shadow taxes — the AGI and MAGI-driven cliffs — are the dominant feature of this
return. Net investment income tax is assessed at $2,449, IRMAA sits above the top tier, and the
SALT deduction is fully phased down to its floor. All three track adjusted gross income of
$791,038 rather than anything about the underlying holdings, but each measures its own
modification of it: IRMAA MAGI is $800,238, being AGI plus the $9,200 of tax-exempt interest,
while NIIT MAGI is $791,038.

| Threshold | This household | Distance |
|---|---|---|
| 37% ordinary bracket — $751,600 | $653,888 ordinary taxable income | $97,712 below |
| 20% capital gains — $600,050 | $713,538 taxable income | $113,488 above; no room beneath |
| Top IRMAA tier — $750,000 MAGI (2026 schedule) | $800,238 IRMAA MAGI | $50,238 above; no higher tier |
| 90% current-year safe harbor — $172,454 | $175,644 withheld | $3,190 above |
| SALT phase-out floor — $10,000 | $10,000 allowed | at the floor |
| NIIT threshold — $250,000 MAGI | $791,038 NIIT MAGI | $541,038 above; $2,449 assessed |
| Additional Medicare threshold — $250,000 | $719,000 wages plus SE income | $469,000 above; $4,477 assessed |
| AMT exemption phase-out — $1,252,700 | $791,038 AGI | $461,662 below |
| Senior deduction phase-out | fully phased out at this AGI | no benefit available |

### The current-year safe harbor cleared by 1.85%

Withholding of $175,644 exceeded the 90%-of-current-year requirement of $172,454 by $3,190, and
the return correctly shows no penalty. At 1.85% above the requirement the cushion is thin, and the
return package's four 1040-ES vouchers totaling $35,136 are sized to the 110%-of-prior-year prong
— $210,777 against the $191,615 liability — on the assumption that withholding continues at the
2025 level.

### Victoria's first Medicare premium year is priced on this return

David reached 65 during 2025. Victoria reaches 65 in 2027, and under the two-year lookback her
first premium year is set by 2025 MAGI of $800,238. The household is above the top tier of the
2026 schedule, and 2027 tiers are not published yet. Once both spouses are enrolled the surcharge
applies per person against the same household MAGI, so the household-level amount roughly doubles
with no change in income.

### $97,712 of the 35% bracket remains below the 37% threshold

Ordinary taxable income is $653,888 — taxable income of $713,538 less the $59,650 preferential
band. The 37% bracket for married filing jointly begins at $751,600 for 2025.

## Strategy signals

**The portfolio reads equity-weighted.** Taxable interest of $4,520 against ordinary dividends of
$19,350 and a $42,500 long-term gain characterizes a portfolio tilted to equities, with the
fixed-income side reported largely as tax-exempt rather than taxable interest.

**The itemized deduction is carried by mortgage interest, not by taxes.** Of $77,500 itemized,
$52,500 is mortgage interest and $15,000 charitable. State and local taxes contribute $10,000
against roughly $45,000 of property tax alone, because the phase-out holds them at the floor.
Additional state or property tax produces no additional federal deduction.

## Checked, nothing observed

- **Self-employment tax against the Social Security wage base.** Schedule SE reports $4,804 on
  $34,000 of net profit — the full 15.3% rate on $31,399 of net earnings, Social Security
  component included. The household carries $685,000 of W-2 wages, far above the contribution and
  benefit base, which appears at first pass to zero out the Social Security component. It does
  not: the wages are David's and the Schedule C is Victoria's. Her base is her own and no wages
  consume it, so the full rate applies to her net earnings and the figure is correct. Attribution
  resolves this check, not magnitude.
- **Capital gain distributions.** $225,000 of proceeds less $190,500 of basis is $34,500, plus the
  $8,000 Schedule D line 13 distribution, reconciling to the $42,500 net long-term gain reported.
  This is the check most likely to misfire, and here it ties.
- **Rollover and QCD treatment.** No Form 1099-R appears on the return, so neither check applies.
- **Zero-basis sales.** All securities sales carry basis; none is reported at zero.
- **Form 8606 per spouse.** No IRA basis or conversion activity appears for either filer, so
  neither return needs one.
- **Real estate professional election.** Schedule E activity is present, but $685,000 of W-2 wages
  makes the material participation hours test implausible for either filer.
- **Section 199A.** No deduction was claimed on the business or rental income. Above the 2025
  threshold a non-service business is limited to the greater of 50% of W-2 wages paid or 25% of
  wages plus 2.5% of unadjusted basis; Schedule C reports no wage expense, so the limitation
  produces nothing and the absence is correct.
- **Enhanced senior deduction.** David qualifies on age, but the deduction phases out well below
  $791,038 of AGI.
- **Alternative minimum tax.** AGI sits $461,662 below the exemption phase-out threshold, with no
  significant preference items on the return.
- **Qualified dividend ratio.** $17,150 of $19,350 in ordinary dividends is qualified — 88.6%,
  which is high and gives no asset-location signal.
- **Tax-exempt interest and IRMAA.** The $9,200 of tax-exempt interest counts toward IRMAA MAGI,
  but the household is already above the top tier, so it does not affect tier placement.

## Limits of this review

These are observations about where the return sits and what it omits, not modeled outcomes — no
figure here projects what a change would produce. The review reads one filed federal return and
the accompanying state summary; account balances, basis outside Schedule D, and holdings are
outside it.

The lookback covers 2025 only, so no year-over-year comparison was possible. Occupation is not
captured in the extraction, so the consistency check against reported income could not run.
Retirement plan limits are not currently returned by `get_tax_constants`, so the SEP and solo
401(k) question above is a question rather than a sized gap.

Reviewed: Form 1040 and Schedules 1, 2, and 3; Schedules A, B, C, D, E, and SE; Forms 8959 and
8960; the California return summary.

Sources: IRS Rev. Proc. 2024-40 — 2025 inflation-adjusted items; SSA — 2025 contribution and
benefit base; CMS — 2026 Medicare Parts A & B premiums; SSA — 2026 income-related monthly
adjustment amounts.
