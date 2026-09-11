# Detection modes, suppression, and the constants contracts

The checks below come from four practitioner sources: **RTS** (2025, OBBBA-aware, the most current
and the richest in reconciliation checks), **Affiance**, **Pikary** (a CPA/PFS practitioner
column), and **Commonwealth** (2014, stale on values but uniquely good on life-event inference).
The **AICPA** review checklists cover almost exclusively Mode 3.

**Use these sources for detection method, never for values.** Every one of them contains at least
one figure that is now wrong — Commonwealth predates the 2018 changes entirely, the AICPA
editions carry 2018 QBI and standard deduction figures, and Pikary predates the OBBBA. Values come
from `get_tax_constants` for federal figures and `get_state_tax_constants` for state ones.

**No source supplies suppression rules.** Suppression is ours to invent, and it is the difference
between a review an advisor trusts and one they stop opening.

---

## Mode 1 — Reconciliation

*Does the return tie to itself?* → **Anomalies**

RTS is the only source with real reconciliation checks. Reads the extraction form, not the
lookback.

| Check | What fires |
|---|---|
| **Rollover reported as income** | Form 1040 lines 4a/5a carry a gross distribution with a taxable amount on 4b/5b that should be absent. Gross with no taxable amount is a rollover, not income. Commonwealth carries the same check against the pre-2018 line 17a/17b |
| **QCD notation** | A distribution treated as a qualified charitable distribution should carry the "QCD" notation on 4a/4b. Present treatment without the notation, or the notation without the treatment |
| **Form 8606 per spouse** | Was there one on each, and should there have been? A joint return with basis or conversion activity for both filers needs two |
| **$0 basis sales** | Securities sales on Schedule D reported with zero basis — usually missing cost basis rather than a real zero |
| **Capital gain distributions** | Schedule D line 13. Read it directly; never back into it from proceeds and basis |

**Suppression — attribution before aggregation.** This is the rule most likely to produce a
confident false positive, because the arithmetic looks right and only the attribution is wrong.

Most per-person limits are genuinely per person, and the return's totals hide that:

| Item | Scope | The error |
|---|---|---|
| Social Security wage base, and the Social Security component of self-employment tax | **Per individual.** Each spouse files a separate Schedule SE against their own base | Netting one spouse's wages against the other spouse's self-employment income. A wife's Schedule C is fully subject to the Social Security component regardless of her husband's wages |
| Excess Social Security withholding credit | **Per taxpayer** | Crediting a couple whose combined withholding crosses one base |
| Form 8606 | **Per spouse** | Treating one as covering both |
| IRA, HSA, and retirement contribution limits | **Per individual** | Applying a household total against one limit |
| Additional Medicare Tax liability | **Per household** — the exception that aggregates | Assuming it works like the others. Withholding is per job; liability is on combined wages |

Before any check that compares an amount to a ceiling, establish whose income it is. Schedule C
carries a proprietor, Schedule SE is filed per spouse, and W-2s are attributed — the return
supports the attribution, but only if the check reads it rather than working from the lookback's
household totals.

**Suppression — capital gain distributions.** RTS names these as a known frustration source, and
this is the exact false positive from the earlier Riley run. Net long-term gain is
(proceeds − basis) **plus** the line 13 distribution. A check that computes gain from Schedule D
part II alone and compares it to the reported total will fire on every return holding a fund that
distributed. Read line 13 first, then reconcile.

**Suppression — rollovers.** The absence of a taxable amount is the signal, not a small taxable
amount. A partial rollover with a genuine taxable remainder is not an anomaly.

---

## Mode 2 — Expected-but-absent

*Is something missing that should be there?* → **Gaps**, and **Triggers** where the absence has a
threshold consequence

The highest-value mode in the set, and where the "surface it without the advisor digging" value
lives. Affiance states the shape cleanly: no deductible IRA on Schedule 1 **and** no Box 12 code
is itself the finding.

**Named absences (RTS Q19):** self-employed health insurance deduction, IRA contribution, HSA
contribution, SEP or solo 401(k). Each is checked against whether the return shows the income or
coverage that would make it available.

**Credits (RTS Q24):** claimed against the dependents, education costs, and income levels the
return reports.

**Expenses you'd expect and don't see:**

- **Schedule C (RTS Q17):** home office, auto, cellphone. A business with meaningful gross
  receipts and a short expense list is the signal.
- **Schedule E (RTS Q18):** depreciation, maintenance, property taxes. Rental income with no
  depreciation is the sharpest of these.

**Elections never made (Pikary):** clients eligible for the real estate professional election who
never made it — which he reports encountering "too often." Requires Schedule E activity plus an
occupation or time signal.

**ACA premium subsidy alert (committed to a customer, August 2026).** Trigger is deliberately
coarse: **no filer is 65 or older, and the return shows no wage income.** That profile is a
pre-Medicare household living on portfolio or retirement income, whose marketplace premium credit
is driven by a modified AGI they may have room to manage. One advisor quantified a single client
at over $150,000 across four years.

Ship this as a **question in Gaps & Follow-Up**, not a calculation — "no wages and both filers
under 65; is anyone covered through the marketplace?" That framing is what the customer asked for
("even if it's an alert… I don't think you need to build it in the calculation"), and it means the
check runs today: the coarse trigger needs no premium-credit constants. Quantifying the position
against the applicable percentage table needs `PREMIUM_TAX_CREDIT`, which is not in the tool yet.

Suppression: an existing Form 8962 or 1095-A means the household is already in the marketplace and
the question is answered — position it under Triggers instead, or drop it.

### The suppression partner

**Absent is not always missing.** Every absence check needs a precondition established before it
fires:

| Absence | Fires only if | Foreclosed by |
|---|---|---|
| HSA contribution | HDHP coverage is plausible | Medicare enrollment — forecloses it entirely |
| IRA contribution | Income within the applicable phase-out, or non-deductible basis plausible | — |
| SEP / solo 401(k) | Self-employment income present | — |
| SE health insurance | Self-employment income and no employer coverage signal | Employer coverage |
| Section 199A | Above the threshold, the wage and basis limitation can correctly produce nothing | No wage expense and no unadjusted basis |
| Real estate professional election | Schedule E activity plus a time or occupation signal | W-2 occupation inconsistent with the hours test |

Where the precondition cannot be established from the return, the item is a **question in Gaps &
Follow-Up**, not a finding.

---

## Mode 3 — Threshold proximity

*Where does the household sit against a cliff or band edge?* → **Triggers**

**"Shadow taxes"** is the advisor-facing name for the AGI/MAGI-driven cliffs — net investment
income tax, IRMAA, and the phase-outs. Adopt the term. It is legible to an advisor in a way that
naming each surtax separately is not, and it groups items that share a cause.

**The shared cause is AGI, not a shared number.** IRMAA adds tax-exempt interest back and NIIT does
not, so the same household has an IRMAA MAGI and a NIIT MAGI that differ by exactly that amount,
and the phase-outs modify AGI differently again. Every MAGI figure names the provision it belongs
to, and the grouping is never written in a way that reads as one measure applied to several
thresholds — that is the presentation half of computing each MAGI separately.

Two kinds of threshold warrant different margins. **Cliffs** — IRMAA tiers, the safe-harbor
requirement — produce a discontinuous jump, so a wider margin is justified. **Band edges and
phase-outs** change a rate at the margin, so a tighter one, because being near a bracket edge is
the ordinary condition of having income.

Report thresholds the household is far from as well. A threshold checked and distant belongs in
the positions table.

**State thresholds.** A graduated state's band edges are measurable the same way, taking the ladder
from `get_state_tax_constants` and measuring it on state taxable income from the state return —
never on federal taxable income, which is a different figure computed under different rules. Check
`structure` first: a flat state has no band edge and a no-income-tax state has no threshold, and
neither is a distance the positions table can carry.

**Suppressions.** A household past the last tier has no next tier to approach. Bracket distance is
measured on ordinary taxable income, not total. A threshold whose measure the return does not
support is unavailable, not distant.

**Degradation.** Bracket distances need `bracketDistribution` from the lookback report, and that
is the first thing dropped when the response is condensed under its character budget. Check
`note`. A condensed response supports the shadow-tax positions — those run off AGI and MAGI, which
survive in `federal.keyFigures` — but not band-edge distances, and those go under Limits rather
than being reconstructed from the rate summary.

---

## Mode 4 — Composition ratio

*What does the mix reveal that the totals don't?* → **Triggers**, **Strategy signals**

Underused and cheap. Reads the mix, not the magnitude.

- **Qualified to ordinary dividend ratio (Pikary).** A low ratio signals an asset-location
  problem — dividend-producing holdings sitting in the wrong account type.
- **Interest income relative to dividend income (AICPA).** Characterizes the underlying portfolio
  without seeing it.
- **Short-term relative to long-term realized gains.**
- **Itemized composition where a category is capped or floored.** A household held to the SALT
  floor has a deduction mix that behaves differently from its face value.

**The methodological correction (Pikary), and it is a hard rule:** *marginal rate cannot be
inferred from income magnitude.* A high-cash-flow return can sit at a low marginal rate where NOL
carryforwards, suspended passive losses, or credit inventory have sheltered the income, and an item
written on the assumption that large income means a top bracket is wrong exactly where it matters.
Compute the rate from the return's actual position, carryforward inventory included.

**Holdings are out of scope.** Tax-exempt interest is read as a figure that feeds IRMAA and NIIT
MAGI, and nothing further: no item characterizes the municipal position, computes a tax-equivalent
yield, or weighs tax-exempt against taxable holdings. That is portfolio advice rather than a return
observation, and it turns on in-state exemption and per-bond facts that neither constants tool
carries.

---

## Mode 5 — Cross-field consistency

*Do two fields that should agree, agree?* → **Anomalies**

RTS Q33 is the model: **occupation reported as "retired" alongside Schedule C income.** Cheap,
specific, and it catches real errors.

Others in the same shape: filing status inconsistent with the household composition implied
elsewhere; tax-exempt interest present with no corresponding Schedule B detail; a state return
inconsistent with the residence the federal return implies.

**The caveat that shapes the mode.** Most of Mode 5 in RTS — Q1, Q4, Q5 — reconciles the return
against CRM records the skill does not have. Those checks cannot fire as anomalies. **They become
questions in Gaps & Follow-Up.** The PRD's note about sourcing supplementation from the estate
side may cover some of them via contact cards; until it does, treat them as questions.

---

## Mode 7 — Life-event inference

*What does this line imply about their life?* → **Gaps & Follow-Up**

Commonwealth is stale on values everywhere else but uniquely good here:

- **Points paid** indicate a property purchase during the year.
- **Interest from many institutions** indicates scattered accounts.
- **High real estate taxes** indicate multiple properties or a high-end residence.

Extend the same shape to ages reaching a threshold window, wage income ending or dropping
sharply, filing status change, and a dependent aging out.

**These are inferences, not findings.** They belong in Gaps & Follow-Up **phrased as questions**.
The question form is what keeps an inference from reading as an assertion about the household —
"the return shows $45,000 of real estate taxes; is there a second property?" rather than "the
client owns multiple properties."

---

## The coverage spine — where to look

The modes above are *how* to look. The AICPA checklist (Chris Benson, CPA/PFS, PFP Section, 2021)
supplies *where*: twelve sections walked against the return side by side. Use it as the walk
order. Coverage does not emerge from good checks — it has to be enumerated, which is why RTS
closes with "have you reviewed ALL of the pages."

**Triage every checklist prompt into one of three classes.** Most of the checklist is not a check,
and treating it as one is how a review turns into a questionnaire:

| Class | Disposition |
|---|---|
| Answerable from the return | A check, run under whichever modes apply |
| Answerable only with data the return doesn't carry | A question in **Gaps & Follow-Up** |
| Advice, discussion, or needs analysis | Out of scope. "Discuss life insurance planning," "explain ownership options," "assist with risk tolerance" — the checklist was written for a CPA in a client meeting, not for a return review |

Stops 13 and 14 are additions. The checklist predates the OBBBA and has **no payments or
withholding stop at all**, which is where several Mode 1 checks and the safe-harbor position live.

| # | Stop | Modes | Return-answerable checks | Becomes a question |
|---|---|---|---|---|
| 1 | Demographics, dependents, filing status | 5, 7 | Filing status; dependent count and ages; occupation against the income sources reported (RTS Q33); filing status change | Estate documents; beneficiary designations; divorce; occupation-driven coverage needs |
| 2 | W-2 income | 1, 2 | Box 12 deferral codes — absent with wage income is the Affiance finding; **Box 12 code W (employer HSA contributions), which is the HDHP precondition signal**; Box 4 against the wage base *per individual*; Box 5 against Box 6 | Employer match maximized; 83(b); option exercise timing; group life, disability, LTC coverage |
| 3 | Additional income / adjustments | 1, 2, 5, 7 | 1099-R gross against taxable (4a/4b, 5a/5b) for rollovers; QCD notation; RMD taken where age requires; 6a/6b against provisional income; Schedule 1 adjustments — IRA, SEP/solo 401(k), SE health insurance, HSA; wage discontinuity | Former-employer plan consolidation; which accounts to draw from; advisor fees paid from the IRA |
| 4 | Schedule B | 2, 4 | Interest against dividend composition; qualified against ordinary dividend ratio; tax-exempt interest and its shadow-tax consequence; foreign account boxes and the FBAR / Form 8938 trigger | Custody; titling; investment policy statement; concentration; reinvestment on or off; emergency reserve |
| 5 | Self-employment income | 1, 2 | Net profit with no retirement plan deduction; no SE health insurance; expenses absent (home office, auto, cellphone); QBI claimed or not against the wage and basis limitation; **Schedule SE per spouse against that spouse's own wage base** | Entity structure; succession; family employment; S-corp shareholder Medicare premium reimbursement |
| 6 | Schedule D | 1, 2, 3, 4 | Line 13 capital gain distribution reconciliation; zero-basis sales; loss carryforward present or exhausted; short against long-term mix; trading volume; position against the 0/15/20 ceilings and room beneath | §1202 eligibility; rebalancing intent |
| 7 | Schedule E | 2, 5 | Rental income with no depreciation; passive against non-passive characterization; real estate professional election absent, with the hours-test suppression; flow-through K-1 with no QBI; trust K-1 against the estate and trust threshold | Property ownership and titling; other-state property; §469 grouping elections; insurance reasonableness |
| 8 | Itemized deductions | 2, 3, 4 | SALT against the cap and phase-down; charitable against the AGI ceilings and the 2026 floor; medical against the floor; mortgage interest against the acquisition limit; investment interest carryover with no §163(d)(4)(B) election | Charitable intent; donor-advised funds and CRTs; appreciated-securities gifting; refinance |
| 9 | AMT | 2, 3 | Liability present; exemption phase-out position; minimum tax credit carryforward on Form 8801 and whether it is being used | ISO exercise planning |
| 10 | Credits | 2 | Education credits against dependents and tuition reported; energy credits; foreign tax credit carryover and the Schedule A against Form 1116 election | Expected education costs |
| 11 | State tax | 3, 5, 7 | Residence against property tax and Schedule E locations; multiple state filings; federal standard against state itemize divergence; the state's `structure` against what the return reports, since tax paid to a state that levies none is an anomaly and a graduated state supports a band-edge distance | Residency change intent; state estate or inheritance exposure; how a destination state treats retirement and Social Security income, which the state table does not carry |
| 12 | Other considerations | — | None. This is the checklist's framing — multi-year planning, timing, legislative change — and it is why the review exists rather than a stop on it | — |
| **13** | **Payments, withholding, safe harbor** *(addition)* | 1, 3 | Form 8959 Part V reconciliation; excess Social Security credit; withholding and estimated payments against the applicable prong, and the cushion | — |
| **14** | **Schedule 1-A and post-OBBBA items** *(addition)* | 2, 3 | Senior deduction, tips, overtime, and car loan interest against their phase-outs | — |

`## Checked, nothing observed` reports against this spine. Without it, that section lists what
happened to get examined rather than what should have been, and the completeness line has nothing
to be complete against.

### Staleness in this edition

The 2021 edition. Take the section taxonomy and the return-answerable prompts; **discard every
value and several whole prompts**:

- *"Advanced child tax credit payments and the pros and cons of opting out"* — 2021 only.
- *"If no RMD was taken due to the 2020 RMD waiver"* — dead.
- *"RMD... age 72, under the Secure Act"* — now 73 under SECURE 2.0.
- *"Sunset of the higher exemption in 2025 or the potential changes to the TCJA rules under the
  Biden administration"* — the sunset did not happen. The OBBBA made the exclusion permanent, so
  the use-it-or-lose-it urgency driving this prompt is not merely stale but **actively
  misleading**, and any gifting item built on it must be suppressed.
- *"The maximum $10,000 state tax deduction under TCJA"* — superseded by the larger cap with a
  high-income phase-down to a $10,000 floor.
- *"Section 199A 20% pass-through deduction under the TCJA"* — now permanent, with mechanics
  changed for 2026.

The QCD age of 70½, the 21% corporate rate, and the elimination of Roth recharacterization are
still current. That mix — mostly stale, partly not — is the reason values come from
`get_tax_constants` rather than from any document.

---

RTS closes with Q36 — *"Have you reviewed ALL of the pages of the return?"* — and Q37 — *"Is there
anything else that doesn't feel right?"* Both are worth borrowing structurally.

Q36 becomes the completeness line: state which schedules and forms were read. Q37 is why
"Checked, nothing observed" exists as a section — it forces the pass to be explicit about what it
looked at and found clean, rather than letting silence carry two meanings.

---

## Suppression rules

The four gates in the skill decide whether an item may render at all. These are the specific
recurring false positives, each a shape that has reached an advisor before. An item matching any of
them does not render.

- **A constant in the payload is never a reason to comment.** The tool scopes sections broadly, so
  the response deliberately contains more than is relevant. Every item traces to a condition on
  *this* return.
- **NIIT is a lesser-of tax.** MAGI above the threshold with no net investment income produces
  nothing. Both legs required.
- **A contribution limit is the base plus every catch-up the filer qualifies for.** Comparing a
  contribution against a base limit alone manufactures excess contributions that don't exist.
- **Entity type is read, never inferred.** Self-employment income reported on Schedule C with SE
  tax paid is not an S corporation, and no S-corp item may attach to it.
- **Additional Medicare Tax under-withholding is expected mechanics**, not an error, for dual
  earners each below the per-job trigger.
- **Excess Social Security withholding is per taxpayer, not per return.**
- **An indexed threshold moves on its own.** Check the threshold before calling a year-over-year
  crossing a change in the household.
- **Already past the last tier is not proximity.**
- **A state without a ladder has no band edge.** In a flat state the rate applies from the first
  dollar, and in a no-income-tax state there is no threshold at all. Neither is a distance of zero,
  and neither belongs in the positions table.
- **No holding-level commentary.** Tax-exempt interest is read as a figure on the return that feeds
  IRMAA and NIIT MAGI. Whether the underlying bonds suit the household is portfolio advice, not a
  return observation: no item characterizes the municipal position, computes a tax-equivalent
  yield, or weighs tax-exempt against taxable holdings.
- **A one-time event stays one-time.** Where the return shows a business sale, large distribution,
  or other non-recurring item, no forward-looking statement treats it as the ongoing position.

Two shapes account for most of what gets through. **Asserting absence without reading** is the
first, and Mode 2 is where it lives: its output is a claim about something that *isn't* there, and
an unpopulated extraction field looks identical to a genuine zero. Every Mode 2 item names the line
it read and states that the line is empty; where the field wasn't returned, the item is
*unavailable* rather than a finding. **Applying a rule the household doesn't qualify for** is the
second. Ages, entity type, coverage, and enrollment are all knowable from the return or from
`household`, and this shape never needs better tax knowledge to prevent — only a precondition
check.

Severity multiplies both. A wrong item does damage in proportion to how alarming it sounded, which
is why the prohibition on severity labels, urgency, and naming a form to file sits in the skill's
`## Critical` rather than in Presentation. It does not lower the error rate; it caps the cost of
the errors that get through.

---

```
get_tax_constants({ taxYear, filingStatus?, sections? })
```

Pass `taxYear` from the extracted return and pass `filingStatus` — every per-status table narrows,
which removes the chance of reading the wrong column. Requesting everything is about 15 KB.

| Section | Used by |
|---|---|
| `ORDINARY_BRACKETS` | Mode 3 band-edge distance; the computed marginal rate |
| `STANDARD_DEDUCTION` | Mode 3 itemized-vs-standard margin; the age-65 addition the lookback omits |
| `SALT_CAP` | Mode 3 phase-out position; Mode 4 deduction composition |
| `CAPITAL_GAINS` | Mode 3 room beneath the 0% and 15% ceilings |
| `SURTAXES` | Mode 3 shadow taxes — NIIT and Additional Medicare |
| `SAFE_HARBOR` | Mode 3 cushion above the applicable prong |
| `IRMAA` | Mode 3 tier distance and the premium year this return sets |
| `SENIOR_DEDUCTION` | Mode 2 whether it survives the phase-out (2025–2028) |
| `ALTERNATIVE_MINIMUM_TAX` | Mode 3 exemption phase-out position |
| `QUALIFIED_BUSINESS_INCOME` | Mode 2 absence check; the wage and basis limitation above the threshold |
| `CHILD_TAX_CREDIT` | Mode 2 credit absence where dependents are claimed |
| `EARNED_INCOME_CREDIT` | Rarely reached in an advisory book |
| `GIFT_AND_ESTATE` | Mode 7 where a life event implies gifting |

Read each section's `note`. Two matter more here than in the executive summary: **SALT is not a
$10,000 cap for a high earner** — it is the larger figure reduced by a phase-out and floored — and
**bracket bands are taxable income with preferential income stacked above ordinary**.

**IRMAA.** The tool resolves the two-year lookback itself. It returns `premiumYear`, `published`,
and `tiersShownForPremiumYear`. When `published` is `false`, say which year you measured against
and note the thresholds may shift. Never present the filing year's own tiers as the household's
future tier.

**Graceful degradation.** An uncovered year returns no `constants` — instead an `error`,
`taxYearsCovered`, `searchInstead`, and `sourcePrecedence`. Search those and say which source you
used. If a figure still cannot be verified, drop the item and record it under Limits. Do not
estimate and do not fall back on a remembered value.

When sources disagree, the IRS revenue procedure wins. The correct value can look like the typo —
the 2026 section 199A threshold really is $25 higher for married filing separately than for single
and head of household, which inverts the usual relationship. Do not normalize a figure from this
tool because it looks wrong.

---

```
get_state_tax_constants({ taxYear, states?, filingStatus? })
```

The state counterpart, holding everything `get_tax_constants` deliberately does not. Same rule:
never a state rate or threshold from memory. Pass `states` (postal codes or full names) or the
result is a one-line summary of all 51 jurisdictions instead of brackets. `filingStatus` is
`SINGLE` or `MARRIED_FILING_JOINTLY` only.

Read `structure` before the brackets — it decides which checks can fire at all:

| `structure` | The brackets are | What that supports |
|---|---|---|
| `GRADUATED` | A marginal ladder of thresholds, not bands; the top rate runs without a ceiling | Mode 3 band-edge distance, measured on state taxable income |
| `FLAT` | One rate from the first dollar, carried as a single entry at $0 | No band edge exists. Report the flat rate; a Mode 3 state distance is not available, it is inapplicable |
| `NONE` | Empty. The state levies no income tax | Mode 5 against a return that nonetheless reports state income tax paid, or a residence the federal return implies elsewhere |
| `NARROW_BASE` | One rate on a base named in `taxBase` — New Hampshire interest and dividends, Washington capital gains | Mode 4 composition where the household holds that base. It implies nothing about wages, and the `standardDeduction` is that base's own exclusion |

Even a graduated ladder is not reliably marginal: footnote markers can flag recapture, where
crossing a threshold applies the top rate to all income. The markers travel with the row and the
footnote text does not, so an item that turns on one says the caveat exists and points to the
source.

Four limits ship with every response:

| Limit | Consequence for an item |
|---|---|
| No married-filing-separately or head-of-household column | Never halve or double the joint figure — a state item for either status is unavailable, not clean |
| Local income taxes excluded (NYC, Maryland counties, Ohio municipalities) | The state rate is not the household's whole state-and-local position |
| Thresholds run on the state's own taxable income | Never map federal taxable income onto a state ladder; some allowances are credits (`isCredit`) against tax rather than deductions from income |
| No retirement, Social Security, or capital gains treatment | Never infer it from the rate schedule |

**Precedence.** There is no state equivalent of "the revenue procedure wins". The state's own
department of revenue is authoritative, this table is a secondary compilation of transcriptions,
and the client's filed return outranks both for what the household actually has.

**Graceful degradation.** An uncovered year returns `error`, `taxYearsWithFullDetail`,
`searchInstead`, and `sourcePrecedence` in place of figures. A year may instead return
`detailAvailable: "STRUCTURE_ONLY"` — whether the state taxes income and how, carrying no rates or
thresholds. The structure checks above still run; everything else goes under Limits. Borrowing a
threshold from an adjacent year is the failure this shape exists to prevent, and state rates change
often enough that it would land wrong.

---

## Not covered by the constants tools today

Checks specified above that cannot fire, with the federal section each needs. Ordered by how much
of the adopted check set they unblock.

| Proposed section | Constants | Unblocks |
|---|---|---|
| `RETIREMENT_LIMITS` | §402(g), §415(c), SEP and solo 401(k) limits, IRA deduction and Roth contribution MAGI phase-outs, age-50 catch-up (indexed — it changed for 2026), age 60–63 catch-up | **Most of Mode 2.** The RTS Q19 absence list is half unbuildable without it — IRA and SEP/solo 401(k) both need the limits to establish availability |
| `HSA` | Contribution limits, HDHP definitions, age-55 catch-up (flat, statutory — distinct from the age-50 IRA catch-up) | The Q19 HSA check and its Medicare-enrollment suppression partner |
| `QCD` | Annual limit under §408(d)(8), indexed | The Mode 1 QCD notation check above the age threshold |
| `SOCIAL_SECURITY` | Provisional-income thresholds and the 50%/85% tiers; contribution and benefit base with the 6.2% rate | Mode 3 taxability thresholds; Mode 1 excess-withholding reconciliation |
| `SE_HEALTH_INSURANCE` | Deduction limits and the interaction with premium credits | The Q19 self-employed health insurance check |
| `CHARITABLE` | AGI percentage ceilings; the 0.5%-of-AGI floor beginning 2026; the 35% cap on itemized benefit (the 2/37ths reduction); the above-the-line cash contribution deduction available to non-itemizers | Mode 3 on the new floor; Mode 4 on deduction composition. **Flagged in production**: an advisor reported charitable items stated from a superseded regime, where the client needed roughly $3,750 of giving just to clear the floor, would have received only the reduced benefit, and was better served by the above-the-line deduction. Until this section exists, charitable items are unavailable rather than clean |
| `PREMIUM_TAX_CREDIT` | Applicable percentage table, required contribution percentage, poverty guidelines by household size with the above-eight increment and separate Alaska and Hawaii schedules, repayment-cap status | Quantifying the ACA position and Mode 3 on the 400%-of-poverty cliff, which returns for 2026 — the only true cliff in the set where crossing zeroes the benefit. **The coarse alert in Mode 2 ships without this**; only the quantified version is blocked |
| `DEPRECIATION` | Recovery periods and conventions for residential rental | Distinguishing "no depreciation claimed" from "correctly none" in the Q18 check |
| `KIDDIE_TAX` | Unearned-income thresholds | Mode 2 and 3 where a dependent has unearned income |
| `SECTION_121` | Exclusion amounts, unindexed | Mode 7 on a residence disposition |

**`RETIREMENT_LIMITS` and `HSA` are the priority.** Mode 2 is the mode with the most advisor value
and the largest share of the adopted checks, and it is the most blocked. Everything else can wait.

**The state table's gaps are source limits, not sections waiting to be built.** They do not close
by adding a section, because the published compilation never carried them. An item needing one of
these is unavailable rather than pending, and saying so is the whole of the handling:

| Gap | What it blocks |
|---|---|
| Local income taxes | Any item treating the state rate as the household's full state-and-local position — New York City and Yonkers, Maryland counties, Ohio and Pennsylvania municipalities, Oregon transit taxes |
| No married-filing-separately or head-of-household columns | State band-edge distance for a household filing either way, in every state |
| State treatment of retirement, Social Security, and capital gains | Mode 7 relocation and retirement-income items |

## Where the arithmetic sits

Every distance in the Triggers table is computed in the skill: take a constant, take a measure off
the return, subtract. That works, and it is why this skill is buildable today. It is also the only
unbacked arithmetic in the skill, repeated once per threshold.

A tool returning positions rather than raw constants would remove it. Worth considering after the
sections above land — there is little point building a positioning surface over a constant set
that is still missing what Mode 2 needs.
