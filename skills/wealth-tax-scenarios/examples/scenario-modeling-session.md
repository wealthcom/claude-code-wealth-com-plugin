# Example — the Riley household, across all three flows

Synthetic fixture throughout (David & Victoria Riley). No real client. Continues from the
`wealth-tax-executive-summary` session on the same household.

---

## Flow 1 — rolling 2025 forward, then layering a change

**Advisor:** Roll them forward to 2026 — and the $42,500 in gains last year was a one-off, so leave it out.

`get_lookback_report(wid, 2025)` → confirms the $42,500 long-term gain to net out

> Rolling the 2025 return forward to 2026, with that $42,500 gain backed out as a one-time item:
>
> 2026 federal tax comes to $151,200, against $178,340 in the 2025 filed year — the difference is mostly the gain coming out, partly 2026's updated brackets.

`model_tax_scenario(wid, baselineYear: 2025, targetYear: 2026, changes: [{input: "capital_gains_net_long_term", changeBy: -42500}])`

*One-time item asked about and netted out explicitly, not rolled forward silently. Both years' figures given so the advisor can see what moved.*

---

**Advisor:** Now layer a $200k Roth conversion on top of that 2026 picture — what does it cost them?

`model_tax_scenario(wid, baselineYear: 2025, targetYear: 2026, changes: [{input: "capital_gains_net_long_term", changeBy: -42500}, {input: "roth_conversion_taxable", changeBy: 200000}])`

> With the $200k conversion added on top of the 2026 baseline we just built, federal tax rises to $221,400 — $70,200 more than that 2026 picture, and it also pushes their ordinary marginal bracket from 32% to 35%.

*Reported as a delta against the just-built baseline, not a bare new number — per the Critical rule on layered changes.*

---

## Flow 2 — building from assumptions, and a real gap

**Advisor:** What would a couple making about $400k owe for 2026 — two kids, standard deduction?

`get_tax_calculation_inputs(taxYear: 2026, filingStatus: null)`

> To run that I need a filing status and each spouse's age — and I'll take $400k as an approximate wage figure unless you want to split it between them. State, too, if you want a state figure alongside the federal one.

**Advisor:** Married filing jointly, both 45, call it Texas, and yes about $400k is right, split evenly.

`get_tax_calculation_inputs(taxYear: 2026, filingStatus: "married_filing_jointly", primaryFilerAge: 45, spouseAge: 45, dependentAges: [9, 12], values: [...])` → nothing required still missing
`run_tax_calculation(taxYear: 2026, filingStatus: "married_filing_jointly", state: "TX", primaryFilerAge: 45, spouseAge: 45, dependentAges: [9, 12], values: [{input: "wages_taxpayer", amount: 200000, approximate: true}, {input: "wages_spouse", amount: 200000, approximate: true}])`

> On $400k split evenly between two W-2 earners, standard deduction, two kids, filing jointly in Texas with no state income tax: federal tax comes to about $68,900 — call it approximate since the wage split itself was approximate. That's a 17.2% effective rate on $400,000, with a 24% marginal bracket.

*The hedge is carried as `approximate: true` on the inputs, and the output figure is described the same way — not silently presented as precise.*

---

**Advisor:** Same couple — they also sold a rental this spring. How does that change the year-end liability?

> To price the sale I need: the net sale price and adjusted basis (or the total gain if you have that already), the accumulated depreciation taken on the property (that drives §1250 recapture), how long they held it, whether they have any suspended passive losses on it, what the property brought in as rental income this year, and whether the property's in the same state as their residence.

**Advisor:** Net sale price was $500,000, adjusted basis $350,000. Depreciation taken over the ownership period was $50,000. Held about six years, no passive losses, rental income was close to breakeven this year, and the property's in Arizona, same as where they live.

> That's a total gain of $150,000. Of that, $50,000 — matching the depreciation taken — is unrecaptured §1250 gain, and the remaining $100,000 is ordinary long-term capital gain.
>
> I can price the $100,000 long-term gain portion normally. The $50,000 unrecaptured §1250 piece isn't something this calculator breaks out separately — it's taxed differently from ordinary capital gains, and folding it into the regular gain figure would understate what they'll actually owe. So I can give you a number for the capital-gains portion of the sale, but not a complete year-end figure for the sale as a whole — that recapture piece would need pricing outside what I can run here.

*The gain and its §1250/LTCG split are derived from the raw sale figures — the advisor isn't expected to hand over a pre-split gain. No figure invented for the unsupported portion; the gap is named as a calculator limitation, not vague hedging, and the session stops there rather than quietly substituting an incomplete number as if it were the answer.*

---

## Flow 3 — sharpened by a document, then a pace check

**Advisor:** Here's a recent paystub — married filing jointly, California. Can you get me a 2026 estimate?

`upload_extract_document(wid, fileName: "riley-paystub-aug2026.pdf", fileKind: "pdf")` → `get_extraction_result(wid, vaultId)` (waits) → salary $744,640 annualized, YTD gross $496,427, YTD federal withholding $117,850, YTD CA withholding $38,655

> From the paystub: David's on pace for about $744,640 in wages this year, with $117,850 withheld federally and $38,655 for California so far. I still need Victoria's income and whether they're taking the standard deduction or itemizing before I can give you a full household estimate.

**Advisor:** Victoria makes about $88,000, and they'll itemize again — similar to last year.

`get_tax_calculation_inputs(...)` → satisfied · `run_tax_calculation(2026, "married_filing_jointly", state: "CA", values: [wages from paystub + Victoria's + itemized deduction estimate carried from last year, each tagged with its source])`

> Combined federal tax for 2026 comes to about $228,000, effective rate 27.4% on $832,640 of wages — with $117,850 of David's withholding counted toward it.

---

**Advisor:** Are they on track with withholding for 2026, or will they owe at filing?

> Against that $228,000 projected federal tax, withholding so far is $117,850 with about four months left in the year — at the current pace that annualizes to roughly $176,800, which would fall short by about $51,200 unless withholding increases or they make an estimated payment before year-end.

*The pace figure is explicitly framed as a projection from partial-year withholding, not a final number, and the shortfall is stated in dollars.*
