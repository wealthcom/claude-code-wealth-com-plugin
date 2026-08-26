# Example — the Riley household, tax year 2025

Synthetic fixture throughout (David & Victoria Riley). No real client.

---

**Advisor:** I've got a review with David and Victoria Riley at 2:00 — pull up their latest return and walk me through their tax picture before I go in.

`find_client("Riley")` → resolved · `get_lookback_report(wid)` → 2025 federal + CA return on file
`get_federal_tax_constants([2025])` · `get_state_tax_constants(["CA"], [2025])`

> # Tax Executive Summary — Riley Family — Tax Year 2025
>
> ## Household & filing
> David and Victoria Riley, married filing jointly, California residents. No dependents on this return.
>
> ## Income
> | Source | Amount |
> |---|---|
> | Wages (David) | $612,000 |
> | Wages (Victoria) | $88,000 |
> | Interest | $4,200 |
> | Qualified dividends | $19,500 |
> | Net long-term capital gains | $42,500 |
> | **Total income** | **$766,200** |
>
> ## Deductions
> The Rileys itemized: $10,000 state and local tax (capped), $28,400 mortgage interest, $31,000 in charitable cash gifts. That's above the $29,200 standard deduction for joint filers, so itemizing was the right call.
>
> ## Tax and rates
> | | Amount |
> |---|---|
> | **Total federal tax** | **$178,340** |
> | State tax (CA) | $58,120 |
>
> The federal effective rate is 23.3% — $178,340 on $766,200 of total income. On ordinary income alone (total taxable income less the $42,500 of long-term gains and $19,500 of qualified dividends), the marginal federal bracket is 35%. California's effective rate is 7.6% — $58,120 on the same income base; CA is a graduated state, so a marginal bracket applies too, at 10.3%.
>
> Combined, federal and state tax comes to $236,460 — that figure isn't a summed rate, just the two dollar amounts added together.
>
> ## Capital gains bracket positioning
> The $42,500 in long-term gains sits in the 15% bracket for 2025, with about $304,000 of room before the household would cross into the 20% bracket.
>
> ## IRMAA positioning
> | MAGI tier | Threshold (MFJ) | This household |
> |---|---|---|
> | Tier 1 | $206,000 | — |
> | Tier 2 | $258,000 | — |
> | Tier 3 | $322,000 | — |
> | Tier 4 | $386,000 | Exceeded by $52,700 |
>
> Their MAGI puts them past Tier 4, so the top IRMAA surcharge tier applies for the premium year this return's MAGI determines.
>
> ## Payments and safe harbor
> Federal withholding was $165,000 against $178,340 owed. Because last year's AGI was over $150,000, the safe harbor threshold is 110% of last year's tax, not 100% — on last year's $161,000 federal tax, that's $177,100. Withholding of $165,000 falls short of that by $12,100, so safe harbor wasn't met on withholding alone; estimated payments of $14,000 bring the total to $179,000, which clears it by $1,900.
>
> ## What's in the return
> | Form/Schedule | Covers |
> |---|---|
> | Form 1040 | Main return |
> | Schedule A | Itemized deductions |
> | Schedule B | Interest and dividends |
> | Schedule D | Capital gains |
> | CA 540 | State return |
>
> Want the full line-item return, or should I go into anything above in more detail?

*All eight sections present, in order. The marginal bracket is computed on ordinary income only (Critical rule). The combined figure is dollars, not a summed rate. IRMAA and safe harbor both state the specific threshold reasoning, not just a pass/fail.*

---

**Advisor:** what would it cost them if David converts $200k to a Roth this year?

> That's a modeling question — I'll hand this to the scenario tool to run it properly against their 2025 baseline rather than estimate it here. One second.

*Explicit handoff to `wealth-tax-scenarios` rather than reaching for a calculation tool inside this skill.*

---

**Advisor:** can you pull up their 2019 revocable trust while we're at it?

> That's an estate document rather than a tax return — I'll switch over to look at the trust itself rather than give it a tax-style summary.

*Redirect to `wealth-estate-document-deep-dive` rather than forcing a tax-shaped answer onto a non-tax document.*
