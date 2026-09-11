# Checks, planning rules, suppression, and the tool contracts

## Evidence model

Every material fact keeps its provenance:

- **Return-confirmed** — read from the filed return or its extraction.
- **Supporting-document** — read from a W-2, K-1, paystub, brokerage statement, plan document, etc.
- **Advisor-provided** — supplied in the conversation.
- **Unknown** — needed to evaluate an item but not established by the available sources.

These describe provenance, not confidence. The tooling gives line-level precision for a full
return and less for general documents — do not treat every document reading as equally
authoritative. When sources disagree, present the conflict and ask.

---

## Integrity pass — the checks

### Mode 1 — Reconciliation and consistency → Return anomalies

Reads the extraction form, not the lookback. Three layers, all high-value and low-drag: each
**reads an amount and matches it**, and none recomputes a subtotal from its components — that is a
full form audit, deliberately out of scope for v1. Every check is tagged to its source.

**Internal ties — does a form hang together?**

| Check | What fires | Source |
|---|---|---|
| Rollover reported as income | 1040 4a/5a gross with a taxable amount on 4b/5b that should be absent — gross with no taxable amount is a rollover
| QCD notation | A distribution treated as a QCD should carry the "QCD" notation on 4a/4b — treatment without the notation, or the notation without the treatment
| Roth conversion reported | A conversion should appear on 4a/4b **and** Form 8606; one without the other is the flag 
| Form 8606 per spouse | A joint return with basis or conversion activity for both filers needs two
| $0 basis sales | Schedule D securities sales at zero basis — usually missing cost basis, not a real zero
| Capital gain distributions | Schedule D line 13. Net LT gain is (proceeds − basis) **plus** line 13; computing gain from Part II alone fires on every fund that distributed — read line 13 first
| Taxable Social Security | 6a gross vs 6b taxable — the taxable share cannot exceed 85%, and its level should track provisional income

**Inter-form presence — does a triggered form or tax actually appear?** A trigger on one line
implies a form or tax elsewhere; its absence is the finding.

| Trigger | Form/tax that should appear | Source |
|---|---|---|
| Net investment income + MAGI over threshold | Form 8960 (NIIT), Schedule 2 line 12 
| Wages or SE income over threshold | Form 8959 (Additional Medicare), Schedule 2 line 11
| Schedule C net profit | Schedule SE, SE tax on Schedule 2 line 4
| AMT preference items | Form 6251, Schedule 2 line 2
| Total tax vs payments | Withholding, estimated payments, and any underpayment penalty consistent — and withholding actually present on retirement income (IRA/pension/SS)

**Schedule carry map — does each bottom line land in the right place?** A fixed set of
lookup-and-match ties: confirm the schedule total appears on its destination line; do **not**
recompute the total. Line numbers below are 2025 — **verify them against the filing year's forms**,
which renumber (post-OBBBA adds Schedule 1-A).

| Bottom line | Should appear on |
|---|---|
| Schedule C line 31 (net profit or loss) | Schedule 1 line 3 |
| Schedule D line 16 (net gain/loss) | 1040 line 7 |
| Schedule E line 41 (total) | Schedule 1 line 5 |
| Schedule F line 34 (net) | Schedule 1 line 6 |
| Schedule B (interest / ordinary dividends) | 1040 line 2b / 3b |
| Schedule SE line 12 (SE tax) / line 13 (½ deduction) | Schedule 2 line 4 / Schedule 1 line 15 |
| Form 6251 / 8959 / 8960 | Schedule 2 line 2 / 11 / 12 |
| Form 8995 or 8995-A (QBI) | 1040 line 13 |
| Schedule A line 17 (itemized) | 1040 line 12e |
| Schedule 8812 (child tax credit) | 1040 line 19 |
| Schedule 1 line 10 / line 26 | 1040 line 8 / 10 |
| Schedule 2 line 3 / line 21 | 1040 line 17 / 23 |
| Schedule 3 line 8 / line 15 | 1040 line 20 / 31 |

**Suppression for all three layers.**

- **Attribution before aggregation** — the most likely confident false positive, because the
  arithmetic looks right and only the attribution is wrong. Per-person limits the totals hide:

| Item | Scope |
|---|---|
| SS wage base; SS component of SE tax | Per individual — each spouse files a separate Schedule SE against their own base |
| Excess SS withholding credit | Per taxpayer |
| Form 8606; IRA/HSA/retirement limits | Per individual |
| Additional Medicare Tax | Per household — the exception that aggregates |

- **Rollovers:** the *absence* of a taxable amount is the signal, not a small one — a partial
  rollover with a genuine remainder is not an anomaly.
- **A carry tie runs only where the extraction exposes both endpoints.** Where it surfaces only the
  destination, or has already normalized the two into one reconciled figure, the tie is
  *unavailable* — not a finding, and an unavailable check is simply not reported.
- **An absent schedule has no carry to check**, and a schedule can legitimately carry a zero.
  Neither is a finding.
- **A presence check needs its trigger established first** — NIIT only where net investment income
  and MAGI both clear the threshold, AMT only where a preference item exists. A missing form
  without its trigger is not an anomaly.
- **A lone checkbox or boolean flag is not an anomaly.** The three layers read *amounts*; a checkbox
  carries no dollar and boolean OCR is the least reliable capture on the form. A flag that
  contradicts the numeric lines around it — a Form 1040 line 16 Form 4972 lump-sum-averaging box
  marked `true` with no distribution on 4a/4b or 5a/5b, a rollover box alongside a full taxable
  amount — is a probable extraction artifact. It is the *flag* that fails the consistency gate, as
  unreliable data, so it is dropped, never surfaced. A boolean becomes reportable only when a numeric
  line corroborates it; where such a flag would change the tax and cannot be reconciled against the
  amounts, it is a question in Questions to validate, never an anomaly.

**Cross-field consistency — do two fields that should agree, agree?** Folded in here; these also
land in Return anomalies. Occupation "retired" alongside Schedule C income (RTS Q33); a state return
inconsistent with the residence the federal return implies; state tax paid to a state that levies
none; tax-exempt interest with no Schedule B detail. Where the mismatch can only be resolved against
CRM records the skill lacks, it is a **question in Questions to validate**, not an anomaly.

### Mode 2 — Expected-but-absent → Questions to validate, Threshold signals

- **Named absences:** self-employed health insurance, IRA, HSA, SEP / solo 401(k) — each
  against whether the return shows the income or coverage that would make it available.
- **Credits:** against the dependents, education costs, and income the return reports.
- **Schedule C expenses:** home office, auto, cellphone — meaningful gross receipts with
  a short expense list is the signal.
- **Schedule E:** depreciation, maintenance, property taxes — rental income with no
  depreciation is the sharpest.
- **Real estate professional election:** Schedule E activity plus a time/occupation signal.
- **ACA premium subsidy alert:** coarse trigger — no filer 65+, and no wage income. A pre-Medicare
  household on portfolio/retirement income whose marketplace credit is driven by a manageable MAGI.
  Ship as a **question** ("no wages, both filers under 65 — is anyone covered through the
  marketplace?"), not a calculation; quantifying it needs `PREMIUM_TAX_CREDIT`, not in the tool
  yet. Suppress if a Form 8962 or 1095-A is already present.

**Suppression partner — absent is not always missing.** Establish the precondition before firing:

| Absence | Fires only if | Foreclosed by |
|---|---|---|
| HSA contribution | HDHP coverage plausible | Medicare enrollment |
| IRA | Income within phase-out, or non-deductible basis plausible | — |
| SEP / solo 401(k) | Self-employment income present | — |
| SE health insurance | SE income and no employer-coverage signal | Employer coverage |
| §199A | Above the threshold, the wage/basis limitation can correctly produce nothing | No wage expense and no unadjusted basis |
| Real estate professional | Schedule E activity + time/occupation signal | Occupation inconsistent with the hours test |

Where the precondition can't be established from the materials, the item is a **question**, not a
finding.

**Employer-plan participation is not on Form 1040.** Elective deferrals and employer contributions
to a 401(k) or 403(b) live on the W-2 — Box 12 (codes D, E, G, S, AA, …) and the Box 13 Retirement
plan indicator — never on the 1040, which shows only the above-the-line IRA, SEP, and solo-401(k)
adjustments on Schedule 1. So with wages present but no W-2 in the materials, whether either spouse
already defers, and how much capacity is left, is *unknown*: a contribution-capacity conversation
ships as a question (or a **Where more is needed** gap naming the W-2), never as an asserted absence
of contributions.

### Mode 3 — Threshold proximity → Threshold & timing signals

**Shadow taxes** — the AGI/MAGI-driven cliffs (NIIT, IRMAA, phase-outs). The shared cause is AGI,
**not a shared number**: IRMAA adds back tax-exempt interest and NIIT does not, so the same
household has an IRMAA MAGI and a NIIT MAGI that differ by exactly that. Every MAGI names its
provision.

**Only surface a threshold when the household is close to it or already over it.** A threshold the
household sits comfortably clear of generates no alert — it is not reported as an item and does not
enter the positions table. "Close" means near enough that a plausible near-term change in the
household's income or MAGI could cross it; "over" means the household is currently subject to the
provision — a surtax being assessed, a benefit phasing out.

The width of "close" depends on the threshold type:

- **Cliffs** (IRMAA tiers, the safe-harbor requirement, the ACA 400%-of-poverty edge) jump
  discontinuously, so they warrant a **wider** band — surface them while the household is still
  approaching, since crossing is costly and often manageable a year ahead.
- **Band edges and phase-outs** (ordinary and capital-gains brackets, SALT, AMT exemption, senior
  deduction) change a rate only at the margin, so they warrant a **tighter** band — sitting some
  distance below a bracket edge is the ordinary condition of having income and is not, by itself,
  an alert.

**Over with no remaining lever is a position, not an alert.** A household already past the top
IRMAA tier has no next tier to approach; note the position where it carries a live consequence, but
do not frame it as proximity ("already past the last tier is not proximity").

**A preferential-rate breakpoint carries its consequence both ways.** Below the 0% capital-gains
breakpoint with qualified dividends or long-term gains present is a harvesting signal. Above it, say
what the household actually pays — the qualified dividends and long-term gains bear 15%, or 20%
above the upper breakpoint — with the dollars that rate falls on, not merely that no 0% room
remains.

**State thresholds** are measured on state taxable income from the state return, never federal.
Check `structure` first — a flat state has no band edge, a no-income-tax state no threshold.

**Degradation:** bracket distances need `bracketDistribution`, the first thing dropped when the
response is condensed. Check `note`; shadow-tax positions still work (they run off AGI/MAGI), while
a band-edge distance you can't compute is simply not surfaced — or goes under Where more is needed
if it blocks an applicable insight.

### Mode 4 — Composition ratio → Threshold signals, Planning opportunities

Reads the mix, not the magnitude.

- Qualified-to-ordinary dividend ratio (low ratio → asset-location signal).
- Interest relative to dividend income; short- vs long-term realized gains.
- Itemized composition where a category is capped or floored (a SALT-floored household's mix
  behaves differently from face value).

**Hard rule:** marginal rate cannot be inferred from income magnitude — NOL
carryforwards, suspended passive losses, or credit inventory can put a high-cash-flow return at a
low marginal rate. **Holdings are out of scope:** tax-exempt interest is read only as a figure that
feeds IRMAA MAGI — no tax-equivalent-yield or municipal-suitability commentary.

---

## Planning pass — trigger families and conditional rules

The planning pass synthesizes facts across sources to surface conversations. A family renders only
when client-specific facts activate it; high income or net worth alone never does — and a large tax
bill is not itself an activating fact. The state-residency and relocation family, in particular,
needs a mobility signal (intent, an out-of-state property, a part-year or multi-state return, a
wage-end or retirement transition), not merely state tax on a big gain. Life events
are one family, not a separate mode — points paid → a property purchase; interest
from many institutions → scattered accounts; high real estate taxes → multiple properties. Extend
the shape to an age reaching a threshold window, wage income ending, a filing-status change, a
dependent aging out. These are inferences → **phrased as questions** where they can't be confirmed.

**Conditional-observation rules (the v1 catalog).** Each is genuinely conditional — it fires only
when the client's extracted data crosses the threshold, never as a static reminder that appears on
every report. Suppress any always-fire "reminder."

| Rule | Fires when | Needs 
|---|---|---|---|
| NIIT exposure | MAGI over the NIIT threshold **and** net investment income present (lesser-of) | MAGI, filing status, NII
| Backdoor-Roth eligibility | MAGI over the direct-Roth phase-out **and** eligible compensation present | MAGI, earned income, filing statu
| Mortgage-interest value | Itemized total sits close to the standard deduction | Schedule A mortgage interest, standard deduction
| HSA contribution vs. limit | Contribution below the annual limit (+ catch-up if 55+), with HDHP coverage plausible | HSA contribution, age, filing status
| Safe-harbor cushion | Withholding + estimates near the applicable prong | Total tax, withholding, prior-year figures 
| Roth-conversion / bracket-fill window | Ordinary taxable income sits materially below the next band edge or IRMAA/NIIT ceiling **and** a low-income window is evident (wages ended or sharply down, filers past 59½ and pre-RMD, pretax assets implied) | ordinary taxable income, bracket/tier ceilings, household ages, 1099-R / retirement signals | Partial — the window and the room ship as a planning opportunity; the conversion sizing is a scenario-modeling handoff |

Where a rule needs a missing constant, the
**conversation still surfaces as a planning opportunity or question** — only the quantified figure
waits. Do not fabricate the figure.

**Roth-conversion / bracket-fill suppression.** Bracket headroom alone never fires it — space below
the next edge is the ordinary condition of not being at the top, and firing on it produces the
generic catalogue. It fires only when headroom coincides with a low-income window that will not
last (wages ended or down, past 59½ and pre-RMD, pretax assets implied). A peak-earning top-bracket
year is the anti-signal, not a candidate: the item keys off *relatively low* income, never income
size. The IRMAA two-year lookback and the NIIT ceiling — the same positions the threshold table
already carries — are the coupled constraint the conversation must respect, since a conversion
lifts MAGI. Sizing (how much, to which ceiling, over how many years) is multi-year and
assumption-heavy, so it is always a scenario-modeling handoff, never a figure stated here; where
the lookback holds one year, say so rather than implying a multi-year read.

**Safe-harbor figures are reproduced, never taken on faith.** The governing requirement is the
lesser of 90% of this year's tax or 100%/110% (high-income AGI) of last year's, built from the
return's total tax, the `SAFE_HARBOR` percentages, and the prior-year return. A "safe harbor" amount
echoed on the lookback, or the sum of the coming year's 1040-ES vouchers, is a planning estimate and
is often not the governing figure — never state a shortfall or a cushion from it. When the
prior-year return isn't in the materials only the 90%-current-year prong is reproducible; if neither
prong can be stood up, ship the return-confirmed underpayment penalty in dollars and route the
withholding question to Questions to validate rather than naming a shortfall.

---

## Suppression rules

The four gates decide whether an item may render. These are the specific recurring false positives
— an item matching any does not render.

- **A constant in the payload is never a reason to comment** — the tool scopes broadly; every item
  traces to a condition on *this* return.
- **NIIT is a lesser-of tax** — MAGI over the threshold with no net investment income produces
  nothing.
- **A contribution limit is the base plus every catch-up the filer qualifies for** — comparing
  against the base alone manufactures phantom excess contributions.
- **Entity type is read, never inferred** — Schedule C with SE tax paid is not an S corp.
- **Additional Medicare under-withholding is expected mechanics** for dual earners each below the
  per-job trigger, not an error. **Excess SS withholding is per taxpayer.**
- **An indexed threshold moves on its own** — check it before calling a year-over-year crossing a
  change in the household.
- **Already past the last tier is not proximity.**
- **A state without a ladder has no band edge** — a flat state's rate applies from the first
  dollar; a no-income-tax state has no threshold. Neither is a distance of zero.
- **A one-time event stays one-time** — a business sale or large distribution is not the ongoing
  position.
- **State tax is not a relocation signal.** The state-residency and relocation family fires only on a
  mobility fact — a stated intent, an out-of-state Schedule E property or a second-home real-estate
  tax, a part-year or multi-state return, a wage-end or retirement transition that frees the
  household to move. High state tax on income, including a large one-time gain, is the "high income
  alone" anti-pattern; absent a mobility fact, residency change ships as a question only where intent
  is present (spine stop 11), never as an opportunity built on the size of the tax.
- **An extraction artifact is not a finding** — a lone low-reliability capture (a checkbox, a boolean
  flag) that conflicts with the numeric lines around it is bad data, not an inconsistency to report;
  reconcile against the amounts and drop the flag (see Mode 1).
- **The 2025 sunset did not happen** — OBBBA made the higher estate exclusion permanent. Any
  gifting item built on use-it-or-lose-it urgency is actively misleading and must be suppressed.

Two shapes account for most of what gets through: **asserting absence without reading** (Mode 2 —
name the line read and empty, or the item is *unavailable*) and **applying a rule the household
doesn't qualify for** (a precondition check, not better tax knowledge, prevents it). Severity
multiplies both, which is why the ban on severity labels, urgency, and naming a form sits in the
skill's `## Critical`: it caps the cost of the errors that get through.

---

## The coverage spine — where to look

Everything before this section in the skill describes what to look for. The modes below explain *how* to look; Walk the stops in order.
Triage every prompt: **answerable from the materials** (a check), **answerable only with data they
don't carry** (a question), or **advice / needs analysis** (out of scope).

| # | Stop | Modes | Return-answerable | Becomes a question |
|---|---|---|---|---|
| 1 | Demographics, dependents, filing status | 1, planning | Filing status; dependent count/ages; occupation vs income sources; status change | Estate docs; beneficiaries; divorce |
| 2 | W-2 income | 1, 2 | Box 12 deferral codes; Box 12 code W (the HDHP signal); Box 4 vs the wage base *per individual*; Box 5 vs Box 6 | Employer match maxed; 83(b); option timing; group coverage |
| 3 | Additional income / adjustments | 1, 2, planning | 1099-R gross vs taxable (rollovers); QCD notation; RMD where age requires; 6a/6b; Schedule 1 adjustments (IRA, SEP/solo 401(k), SE health, HSA); wage discontinuity | Plan consolidation; which accounts to draw from; advisor fees from the IRA |
| 4 | Schedule B | 2, 4 | Interest vs dividend composition; qualified vs ordinary ratio; tax-exempt interest and its shadow-tax consequence; FBAR/8938 trigger | Custody; titling; concentration; reinvestment |
| 5 | Self-employment | 1, 2 | Net profit with no plan deduction; no SE health; expenses absent; QBI vs the wage/basis limit; **Schedule SE per spouse against that spouse's base** | Entity structure; succession; family employment |
| 6 | Schedule D | 1, 2, 3, 4 | Line 13 reconciliation; zero-basis sales; loss carryforward; ST vs LT mix; position vs the 0/15/20 ceilings | §1202; rebalancing intent |
| 7 | Schedule E | 1, 2 | Rental income with no depreciation; passive vs non-passive; real estate professional election (with the hours-test suppression); flow-through K-1 with no QBI; trust K-1 | Property titling; other-state property; §469 grouping |
| 8 | Itemized deductions | 2, 3, 4 | SALT vs the cap and phase-down; charitable vs the AGI ceilings and the 2026 floor; medical floor; mortgage interest vs the acquisition limit; investment-interest carryover | Charitable intent; DAFs and CRTs; appreciated-securities gifting; refinance |
| 9 | AMT | 2, 3 | Liability present; exemption phase-out; minimum-tax-credit carryforward (8801) | ISO or RSU exercise planning |
| 10 | Credits | 2 | Education credits vs dependents/tuition; FTC carryover and the 1116 election | Expected education costs |
| 11 | State tax | 1, 3, planning | Residence vs property tax and Schedule E locations; multiple-state filings; the state's `structure` vs what the return reports (tax paid to a no-income-tax state is an anomaly) | Residency-change intent; state estate exposure; how a destination state treats retirement/SS income |
| 12 | Payments, withholding, safe harbor *(added)* | 1, 3 | Form 8959 Part V; excess SS credit; withholding/estimates vs the applicable safe harbor prong and the cushion | — |
| 13 | Schedule 1-A / post-OBBBA | 2, 3 | Senior deduction, tips, overtime, car-loan interest vs their phase-outs | — |

The spine is the universe the review is *complete against* — walk every stop so coverage is
enumerated, not emergent (four good items and an unopened Schedule E is not a review). But the spine
is not a list to report. The output carries only items that prompt a conversation or a correction; a
stop that was walked and held simply produces no line, and there is no "nothing observed"
enumeration and no recital of everything reviewed — that silence is the brevity the advisor asked
for. The one place a walked stop reaches the page without a finding is **Where more is needed**: when
a stop surfaces an insight that looks applicable but the materials can't fully support it, name it
there against the single missing input.

---

## The constants tools

External federal tax law — brackets, caps, thresholds, tiers, and percentages that are **not** on
the client's return and **not** returned by any other tool. Call the tool; never state one of these
from memory. They re-index every year, and recent statutory changes post-date most training data, so
a remembered value is likely wrong, not merely stale. Most of what follows is the federal tool; the
state counterpart has its own subsection below.

### Call it

```
get_federal_tax_constants({ taxYear, filingStatus, sections? })
```

- **`taxYear`** (required) — the return's tax year, taken from the extracted return, not the
  document name.
- **`filingStatus`** — `SINGLE`, `MARRIED_FILING_JOINTLY`, `MARRIED_FILING_SEPARATELY`, or
  `HEAD_OF_HOUSEHOLD`. Pass it: every per-status table narrows to that one status, which shortens the
  result and removes the chance of reading the wrong column.
- **`sections`** — omit to get everything (~15 KB); narrow it when you need only part.

| Section | Covers |
|---|---|
| `ORDINARY_BRACKETS` | Ordinary income bracket bands |
| `STANDARD_DEDUCTION` | Base amount, plus the age-65 addition for married and unmarried filers |
| `SALT_CAP` | Cap, phase-out threshold and rate, floor, and the MAGI at which the floor is reached |
| `CAPITAL_GAINS` | 0% and 15% ceilings for long-term gains and qualified dividends |
| `SURTAXES` | Net investment income tax and Additional Medicare tax — rates and thresholds |
| `SAFE_HARBOR` | Current- and prior-year percentages, the high-income AGI trigger, de minimis balance |
| `IRMAA` | Medicare Part B and Part D tiers for the premium year this return drives |
| `SENIOR_DEDUCTION` | The enhanced deduction for filers age 65 or older (2025–2028) |
| `ALTERNATIVE_MINIMUM_TAX` | Exemption, phase-out threshold and rate, and the 28% rate threshold |
| `QUALIFIED_BUSINESS_INCOME` | Section 199A threshold, phase-in range, and the 2026 minimum deduction |
| `EARNED_INCOME_CREDIT` | Maximum credit, earned income amount, and phase-out band by child count |
| `CHILD_TAX_CREDIT` | Maximum per child, refundable portion, other-dependent credit, phase-out |
| `GIFT_AND_ESTATE` | Annual gift exclusion, non-citizen spouse exclusion, lifetime basic exclusion |

Across the modes, insights leans on `ORDINARY_BRACKETS`, `CAPITAL_GAINS`, `SURTAXES`, `SAFE_HARBOR`,
and `IRMAA` for the Mode 3 threshold positions; `STANDARD_DEDUCTION` and `SALT_CAP` for the itemized
and phase-out reads; `QUALIFIED_BUSINESS_INCOME` and `SENIOR_DEDUCTION` for the Mode 2 absence
checks; and `GIFT_AND_ESTATE` where a life event implies gifting. Requesting everything is still fine.

### Reading the result

Each section carries its own `note` — read it. Three exist to stop a plausible-sounding but wrong
statement:

- **SALT is not a $10,000 cap for a high earner.** The cap is the larger figure; the phase-out
  reduces it and floors it at $10,000. Say the household is held to the floor by the phase-out.
- **The bracket bands are ordinary taxable income**, with preferential income stacked above ordinary. Measure
  the marginal ordinary bracket on taxable income *less* preferential income, and quote the
  thresholds exactly as returned — the ladder is only as good as the row the marker sits on.
- **The standard deduction on the lookback omits the age-65 addition.** Build the age-adjusted figure
  from the tool's base plus its addition, per qualifying filer.

`sources` maps each returned section to its publisher — use it for the output's **Sources** line.
Figures read off the client's return need no citation; anything from this tool does.

#### IRMAA

The tool resolves the two-year lookback itself: a year's MAGI sets Medicare premiums two years later,
and that year is usually unpublished when the return is filed.

| Field | Meaning |
|---|---|
| `premiumYear` | The year this return's MAGI will set |
| `published` | Whether that year's tiers exist yet |
| `tiersShownForPremiumYear` | The year the returned tiers belong to |

When `published` is `false`, the tiers are the most recent published year's — say which year you
measured against and note the real thresholds may shift. Never present the filing year's own tiers as
the household's future tier.

### Graceful degradation

**Uncovered tax year.** The result has no `constants` — inform the user that you do not have the necessary constants to evaluate.

**A section you need is missing or empty.** Same rule — name what is unavailable, drop the derived
item, keep the rest.

### The state counterpart

```
get_state_tax_constants({ taxYear, states, filingStatus })
```

Same rule — never a state rate or threshold from memory, since states re-index annually and several
are mid-phase-down. Pass `states` (postal codes or full names), or you get a one-line summary of all
51 jurisdictions instead of brackets. `filingStatus` is `SINGLE` or `MARRIED_FILING_JOINTLY` only.

Read `structure` before the brackets. Every bracket reads "this rate applies to income over $X", but
what that amounts to depends on the structure:

| `structure` | The brackets are |
|---|---|
| `GRADUATED` | A marginal ladder of thresholds, not bands; the top rate runs without a ceiling — a band edge to measure against |
| `FLAT` | One rate from the first dollar, carried as a single entry at $0. Report the flat rate — there is no bracket to sit near and no marginal-versus-effective distinction to draw |
| `NONE` | Empty. The state levies no income tax; say that rather than reporting a rate — and a return that nonetheless reports state tax paid is an anomaly |
| `NARROW_BASE` | One rate on a base that is not ordinary income, named in `taxBase` (New Hampshire interest and dividends, Washington capital gains). It implies nothing about wages, and `standardDeduction` is that base's own exclusion |

Four more limits ship with every response, each a way to get a state figure wrong from a correct
table:

| Limit | What to say instead |
|---|---|
| No married-filing-separately or head-of-household column | The table has no column for that status — never halve or double the joint figure; the item is unavailable, not clean |
| Local income taxes excluded (NYC, Maryland counties, Ohio municipalities) | The state rate is not the whole state-and-local position |
| Thresholds run on the state's own taxable income | Never map federal taxable income onto a state ladder; some allowances are credits (`isCredit`) that come off tax, not income |
| No retirement, Social Security, or capital gains treatment | Never infer it from the rate schedule |