# Tax Insights — Kapoor Family — Tax Year 2025

> Reference output. Match this for section order, tone, and level of detail.
>
> COPY THE SHAPE, NEVER THE NUMBERS. The Kapoors are a real household; every figure below is theirs
> alone. Take every figure in a real review from that client's own return, never from here.
>
> The output opens straight on the first finding — no preamble, no household or income recap. This
> household runs two Schedule C businesses, a non-SSTB food business and an SSTB consulting practice,
> which is what makes the 199A and entity-structure items live; read the sections against that shape
> rather than against the specific dollars.
>
> Every section carries content in this one. A household whose reconciliation checks all tie would
> show no `## Return anomalies` section at all — an empty section is omitted, not padded, and the
> review never enumerates the stops it walked and cleared.

## Return anomalies

### Schedule E rental reports no depreciation

The federal return's Schedule E shows the Arlington property (1430 21st St S) rented all 365 days
with $42,000 of rents, $16,000 of mortgage interest, and $34,870 of total expenses — and depreciation
of $0 on Line 18 (return-confirmed). A residential rental held the full year would normally carry an
annual depreciation deduction; its absence understates expenses and overstates the $7,130 net the
return reports, and that overstated income flows through to the net investment income tax. Whether
depreciation was omitted or the property is non-depreciable — land-only or fully depreciated — is not
something the return settles.

## Planning opportunities

### Retirement-plan design against two self-employed incomes

**Why this surfaced.** Both Schedule Cs report $0 on the pension and profit-sharing line (Line 19),
and Schedule 1 Line 16 (SEP, SIMPLE, and qualified plans) is $0, against combined net self-employment
profit of $408,200 — Devin's food business at $178,200 and Priya's consulting at $230,000
(return-confirmed). Self-employment income with no plan deduction is the room; at ages 63 and 61 an
age-weighted defined-benefit or cash-balance design can shelter well beyond SEP or solo-401(k) limits,
and the deferral also lowers taxable income toward the 199A threshold below. Whether a plan is already
held outside this return is not visible here.

**Conversation to have.** Is either business already running a plan held elsewhere? Is the goal
maximum current-year deferral or contribution flexibility, and does either business have non-owner
employees who would have to be covered?

**Useful documents.** Any existing plan documents, payroll reports, and an employee census for each
business.

**Possible next analysis.** Compare solo-401(k), SEP, and cash-balance capacity for each spouse once
the employee and cash-flow facts are confirmed — a scenario-modeling handoff, since the contribution
limits are not returned by `get_federal_tax_constants` today.

### Entity structure against the self-employment tax

**Why this surfaced.** Self-employment tax totals $53,174 — Devin $25,178, Priya $27,996
(return-confirmed). Devin's food-business SE base of $164,568 sits below the $176,100 Social Security
wage base, so an S-corp election there could reach both the 12.4% Social Security and the Medicare
portions on earnings above reasonable compensation, and the W-2 wages it would create support his
non-SSTB QBI wage limitation if income rises. Priya's consulting SE base already exceeds the wage
base, so an S-corp there reaches only the Medicare portion. Which business is the stronger candidate
turns on facts the return doesn't carry.

**Conversation to have.** Appetite for payroll and its administration; what reasonable compensation
each role supports; whether Devin's food business is the stronger candidate given the combined SE-tax
and QBI effect.

**Possible next analysis.** An S-corp-versus-Schedule C comparison for each business, netting payroll
cost against SE-tax savings and the QBI effect.

### QBI deduction sitting on the 199A threshold with an SSTB in the mix

**Why this surfaced.** Taxable income before the QBI deduction is $395,243 — $643 over the $394,600
married-filing-jointly threshold. Priya's Management Consulting is a specified service business with
$0 of W-2 wages, and Devin's food business is non-SSTB, also with $0 of W-2 wages; the $74,449 QBI
deduction is currently held down by the taxable-income limitation rather than by the wage or SSTB
tests (return-confirmed; SSTB status read from the "Management Consulting" description on Priya's
Schedule C). The position is fragile — as income climbs into the $394,600–$494,600 phase-out range,
Priya's SSTB deduction disappears and Devin's becomes wage-limited. The retirement deferral above, by
lowering taxable income back under $394,600, would restore the full deduction.

**Conversation to have.** Confirm Priya's activity is a consulting SSTB, and how close income runs to
the threshold in a typical year.

**Possible next analysis.** Model the QBI deduction under a retirement-contribution scenario that
drops taxable income below $394,600.

### Roth conversion window before RMDs and Medicare

**Why this surfaced.** Both spouses are pre-RMD and pre-Medicare, and the $14,000 IRA distribution
(Line 4a/4b) confirms pretax IRA balances exist (return-confirmed). The 24% bracket shows $96,806 of
room on the return's bracket table before the 32% bracket. The constraint is that the top of the 24%
bracket ($394,600) is the same line as the 199A threshold, and any conversion lifts Devin's IRMAA
MAGI for 2027 — so the usable room is narrower than the bracket alone suggests.

**Conversation to have.** Total pretax IRA and 401(k) balances and any existing basis; tolerance for
lifting this year's MAGI given the IRMAA and 199A interactions.

**Possible next analysis.** Model conversion amounts against the $394,600 line and Devin's 2027 IRMAA
tier.

### HSA funded far below capacity, with a closing window

**Why this surfaced.** The HSA deduction is $2,000 on Schedule 1 Line 13, in a household where both
spouses are over 55 and each catch-up-eligible in their own account, and where Devin's HSA
eligibility ends when he enrolls in Medicare in 2027 (return-confirmed). The gap between $2,000 and
the family limit points to unused capacity before that window closes, though the exact figure turns
on the coverage type — see Where more is needed.

**Conversation to have.** Is the HDHP coverage self-only or family, is each spouse using their own
catch-up, and when will Devin enroll in Medicare?

**Useful documents.** HDHP plan confirmation and current HSA statements.

### Cash charitable gifts while holding appreciated securities

**Why this surfaced.** The $20,000 of charitable giving was made entirely by cash or check (Schedule
A Line 11), while the household holds appreciated long-term securities — Schedule D shows $40,000 of
proceeds against $20,000 of basis, a 50% embedded gain (return-confirmed). Gifting appreciated
long-term shares rather than cash would avoid the gain on the donated shares while preserving the
deduction, and a donor-advised fund would let the household bunch several years of giving into one.

**Conversation to have.** Is the giving recurring, and is there openness to gifting shares in-kind or
funding a donor-advised fund?

## Threshold & timing signals

Two of the Kapoors' shadow taxes — the AGI- and MAGI-driven cliffs — are already live, and each
measures its own modification of income. Net investment income tax is assessed at $1,881: NIIT MAGI
of $445,743 stands $195,743 above the $250,000 threshold, so essentially all net investment income —
interest, dividends, the capital gain, and the passive rental — is exposed at 3.8%. IRMAA runs on a
separate clock: Devin turns 65 in November 2027, and the two-year lookback means this 2025 return
sets his 2027 Part B and D premium off IRMAA MAGI (AGI plus tax-exempt interest) of $445,743, a high
tier. 2026 income will set 2028, and Priya's first premium year — she turns 65 in 2029 — is priced on
2027 income.

| Threshold | This household | Distance |
|---|---|---|
| 199A QBI threshold (MFJ) — $394,600 | $395,243 taxable income before QBI | $643 above |
| 24% → 32% bracket top — $394,600 | $297,794 ordinary taxable income | $96,806 below |
| NIIT threshold — $250,000 MAGI | $445,743 NIIT MAGI | $195,743 above; $1,881 assessed |
| Additional Medicare threshold — $250,000 | $376,973 self-employment earnings | $126,973 above; $1,143 assessed |
| SALT phase-out — $500,000 MAGI | $445,743 MAGI | $54,257 below |

### The $394,600–$500,000 band is crowded

The 199A cliff and the 24%/32% bracket break both fall at $394,600, and the SALT-cap phase-out and the
top IRMAA tier both begin at $500,000. Additional ordinary income in this band is unusually expensive
per dollar — it erodes the QBI deduction on the way up and, past $500,000, phases down the SALT
deduction. It is the constraint behind the Roth-conversion sizing above.

## Questions to validate

**Does either spouse hold a retirement plan outside this return?** Schedule C Line 19 and Schedule 1
Line 16 are both $0, but a plan held elsewhere would not appear here.

**Was rental depreciation omitted?** Schedule E Line 18 is $0 on a property rented all year with a
mortgage — confirm whether depreciation was missed or the property is non-depreciable.

**What is the HDHP coverage type, and are both catch-ups used?** The $2,000 HSA deduction points to
substantial unused room, but the exact gap depends on self-only versus family coverage.

**Are the federal estimates intentionally running to a refund?** $128,000 of estimates with $0
withholding produced an $11,187 federal refund and no penalty — confirm whether applying the
overpayment forward is preferred.

## Where more is needed

**HSA capacity** looks applicable, but the exact unused amount can't be stated here: the contribution
limit is not carried by `get_federal_tax_constants`, and the figure turns on confirming self-only
versus family HDHP coverage and each spouse's catch-up. The coverage type would settle it.
