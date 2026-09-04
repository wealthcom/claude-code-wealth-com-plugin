# `get_tax_constants` and `get_state_tax_constants` — tool contracts and graceful degradation

Most of what follows is the federal tool; the state counterpart has its own section near the end.

External federal tax law: brackets, caps, thresholds, tiers, and percentages that are **not** on
the client's return and are **not** returned by any other tool. Call the tool; never state one of
these figures from memory. They are re-indexed every year, and recent statutory changes post-date
most training data, so a remembered value is likely wrong rather than merely stale.

## Call it

```
get_tax_constants({ taxYear, filingStatus?, sections? })
```

- **`taxYear`** (required) — the tax year of the return being summarized. Take it from the
  extracted return, not from the document name.
- **`filingStatus`** — `SINGLE`, `MARRIED_FILING_JOINTLY`, `MARRIED_FILING_SEPARATELY`, or
  `HEAD_OF_HOUSEHOLD`. Pass it. Every per-status table narrows to that one status, which makes
  the result shorter and removes the chance of reading the wrong column.
- **`sections`** — omit to get everything. Narrow it when you only need part.

| Section | Covers |
|---|---|
| `ORDINARY_BRACKETS` | Ordinary income bracket bands |
| `STANDARD_DEDUCTION` | Base amount, plus the age-65 addition for married and unmarried filers |
| `SALT_CAP` | Cap, phase-out threshold and rate, floor, and the MAGI at which the floor is reached |
| `CAPITAL_GAINS` | 0% and 15% ceilings for long-term gains and qualified dividends |
| `SURTAXES` | Net investment income tax and additional Medicare tax — rates and thresholds |
| `SAFE_HARBOR` | Current-year and prior-year percentages, the high-income AGI trigger, de minimis balance |
| `IRMAA` | Medicare Part B and Part D tiers for the premium year this return drives |
| `SENIOR_DEDUCTION` | The enhanced deduction for filers age 65 or older (tax years 2025–2028) |
| `ALTERNATIVE_MINIMUM_TAX` | Exemption, phase-out threshold and rate, and the 28% rate threshold |
| `QUALIFIED_BUSINESS_INCOME` | Section 199A threshold, phase-in range, and the 2026 minimum deduction |
| `EARNED_INCOME_CREDIT` | Maximum credit, earned income amount, and phase-out band by child count |
| `CHILD_TAX_CREDIT` | Maximum per child, refundable portion, other-dependent credit, phase-out |
| `GIFT_AND_ESTATE` | Annual gift exclusion, non-citizen spouse exclusion, lifetime basic exclusion |

For the executive summary, the sections that matter are `STANDARD_DEDUCTION` (the age-adjusted
figure the lookback omits), `SALT_CAP`, `CAPITAL_GAINS`, `ORDINARY_BRACKETS`, `SURTAXES`,
`SAFE_HARBOR`, and `IRMAA`. Requesting those seven is the normal call. The last five exist for
returns that reach them — a Form 6251 with an AMT liability, a Schedule C or K-1 with a section
199A deduction, a return claiming the child or earned income credit, a Form 709 — and for the
insights and gap-analysis skills. Asking for everything is still fine; it is about 15 KB.

## Reading the result

Each section carries its own `note` — read it. Several notes exist specifically to stop a
plausible-sounding but wrong statement, in particular:

- **SALT is not a $10,000 cap for a high earner.** The cap is the larger figure; the phase-out
  reduces it and floors it at $10,000. Say the household is held to the floor by the phase-out.
- **The bracket bands are taxable income**, and preferential income stacks above ordinary taxable
  income. Measure the marginal ordinary bracket on taxable income *less* preferential income. The
  executive summary now renders `ORDINARY_BRACKETS` and `CAPITAL_GAINS` as the two §6
  bracket-positioning ladders and uses them for the §4 income-tax build (ordinary-bracket tax on
  the ordinary slice + preferential-rate tax on the preferential slice), so quote these thresholds
  exactly as the tool returns them — the ladder is only as trustworthy as the row the marker sits
  on.
- **The standard deduction figure stored on the lookback omits the age-65 addition.** Build the
  age-adjusted figure from the tool's base plus its addition, per qualifying filer.

`sources` maps each returned section to its publisher. Use it to build the summary's **Sources**
line. Figures read off the client's return need no citation; anything from this tool does.

### IRMAA

The tool resolves the two-year lookback itself. A tax year's MAGI sets Medicare premiums two
years later, and that later year is usually not published yet when the return is filed. The
result tells you which case you are in:

| Field | Meaning |
|---|---|
| `premiumYear` | The year this return's MAGI will actually set |
| `published` | Whether that year's tiers exist yet |
| `tiersShownForPremiumYear` | The year the returned tiers belong to |

When `published` is `false`, the tiers are the most recent published year's. Say which year you
measured against and note that the real thresholds may shift — the Riley example shows the
wording. Never present the filing year's own tiers as the household's future tier.

## Graceful degradation

**Uncovered tax year.** The result has no `constants`. Instead it carries an `error`, the
`taxYearsCovered` list, `searchInstead` — the publishers for that figure — and
`sourcePrecedence`. Search those, and say in the summary which source you used. If you still
cannot verify a figure, say so plainly and leave the analysis that depended on it out. Do not
estimate, and do not fall back on a remembered value.

When two sources disagree, the IRS revenue procedure wins. Secondary summaries are transcriptions
and do carry errors.

That last class of error is the one to watch, because the correct value can look like the typo.
The 2026 section 199A threshold really is $25 *higher* for married filing separately ($201,775)
than for single and head of household ($201,750), which inverts the usual relationship and
matches Forms 8995 and 8995-A. Do not normalize a figure from this tool because it looks wrong.

**A section you need is missing or empty.** Same rule: name what is unavailable, drop the derived
analysis, and keep the rest of the summary. The skill's presentation rules already require this —
"If unverifiable, say so plainly."

**State figures never come from this tool, by design.** Statutory state law comes from
`get_state_tax_constants`, below; what the household actually paid its state comes from
`get_lookback_report`, and the return wins where the two disagree.

## The state counterpart

```
get_state_tax_constants({ taxYear, states?, filingStatus? })
```

Same rule as the federal tool — never from memory, since states re-index annually and several are
mid-phase-down. Pass `states` (postal codes or full names), or you get a one-line summary of all 51
jurisdictions instead of brackets. `filingStatus` is `SINGLE` or `MARRIED_FILING_JOINTLY` only.

Read `structure` before the brackets. Every bracket is published as "this rate applies to income
over $X", but what that table amounts to depends on the structure:

| `structure` | The brackets are |
|---|---|
| `GRADUATED` | A marginal ladder of thresholds, not bands; the top rate runs without a ceiling |
| `FLAT` | One rate from the first dollar, carried as a single entry at $0. Say "a flat 4.4%" — there is no bracket to be in and no marginal-versus-effective distinction to draw |
| `NONE` | Empty. The state levies no income tax; say that rather than reporting a rate |
| `NARROW_BASE` | One rate on a base that is not ordinary income, named in `taxBase` (New Hampshire interest and dividends, Washington capital gains). It implies nothing about wages, and the `standardDeduction` is that base's own exclusion |

Even a graduated ladder is not reliably marginal: footnote markers can flag recapture, where
crossing a threshold applies the top rate to all income. The markers travel with the row and their
text does not, so when the answer turns on one, say the caveat exists and point to the source.

Four more limits ship with every response, each a way to get a state figure wrong from a correct
table:

| Limit | What to say instead |
|---|---|
| No married-filing-separately or head-of-household column | The table has no column for that status — never halve or double the joint figure |
| Local income taxes excluded (NYC, Maryland counties, Ohio municipalities) | The state rate is not the whole state-and-local liability |
| Thresholds run on the state's own taxable income | Never map federal taxable income onto a state ladder; the state's deductions and exemptions are its own, and some are credits (`isCredit`) that come off tax rather than income |
| No retirement, Social Security, or capital gains treatment | Never infer it from the rate schedule |

The source is a secondary compilation, so there is no "the revenue procedure wins" here: the
state's department of revenue settles the law, the client's filed return settles the client. An
uncovered year degrades as above; `detailAvailable: "STRUCTURE_ONLY"` means the year carries only
whether the state taxes income and how — use it for that, and borrow no thresholds from an
adjacent year.

## Maintaining the data

The figures live in `src/mcp/tools/taxConstantsData.ts`, not in this skill — the insights and
gap-analysis skills read the same values through the same tool. Adding a tax year means one new
entry there when the IRS revenue procedure lands, and one new IRMAA entry when CMS and SSA
publish. Nothing in this skill changes.

The state figures live alongside them in `src/mcp/tools/stateTaxConstantsData.ts`, generated from
the Tax Foundation's annual export by `src/scripts/generate-state-tax-constants.ts`. A new year
arrives by regenerating that file, which is also how a structure-only year becomes a full one.
