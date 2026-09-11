# Tool contracts — wealth-tax-scenarios

Contract and failure shape only; when and why to call each is in SKILL.md. Never state a tax-law
figure from memory — the constants tools own them.

## The routine calls

| Tool | Returns | Contract |
|---|---|---|
| `find_client` | a book match, else a filer-name match | Honor the match basis — a filer match means a different surname is on file. Empty means unmatched, not absent. |
| `list_tax_years` | per-year verdict: filed / modelable / neither | The verdict picks the path, not the existence of a client record. |
| `extract_tax_document` → `get_extraction_result` | an attachment handle, then its findings | Saves nothing to the profile; a retry does not start a second job. Raises a value's quality, not its origin. |
| `get_lookback_report` | the filed baseline and the reference the modeling tools need | Reads an existing baseline only — never to resolve a household for promotion (that is `find_client`). Every value carries filed origin. |
| `create_tax_scenario` | commits the approved scenario to the profile | Sandbox state is temporary and expires. Commits exactly what was approved; batched confirmation is not confirmation; needs a resolved household. |

## `get_tax_calculation_inputs`

The target year's fields, split into what is required and what is still missing. Re-call it as values
arrive: new information changes the required set — filing status and state of residence in particular
unlock or reshape what else is needed. Required-satisfied is not the same as complete on Path A — the
optional fields the advisor never raised are still absent, and the figure is only as good as what
went in. It describes a free-form calculation exactly as it does a templated one.

## `run_tax_calculation`

Computes or declines — never estimates. A decline is structured: what it could not do, and what would
let it proceed. Report that; do not fill it.

The worksheet can only surface what this returns on success. Confirm the return shape by testing,
then record it here — anything left unchecked is a tool-level gap, not a skill gap:

- [ ] AGI, taxable income, tax before credits, and final liability, as subtotals
- [ ] per-bracket slices (band, width, rate, dollars)
- [ ] the preferential-income build (see below) as its own line, not folded into ordinary tax
- [ ] surtaxes — net investment income tax, additional Medicare — as separate components
- [ ] credit line items
- [ ] the state build, where there is one

## Reading the constants — `get_federal_tax_constants`, `get_state_tax_constants`

Pass the tax year and filing status (plus `states` for the state tool). Never from memory — both
re-index annually and recent statutory changes post-date most training data. On an uncovered year the
result carries no constants but names the publishers to search: report the gap and cite the source;
never borrow a nearby year, another state, or federal rules standing in for state.

The handful that change a scenario's number, and that a correct worksheet has to reflect:

- **Preferential income stacks above ordinary.** Long-term gains and qualified dividends are taxed on
  their own 0/15/20 ladder, sitting on top of ordinary taxable income. Run the ordinary brackets on
  taxable income *less* the preferential slice, then tax that slice on its own ladder — two builds,
  not one bracket run over everything.
- **The standard deduction carries an age-65 addition** per qualifying filer, on top of the base for
  the filing status. The base figure alone understates it for a household 65 or older.
- **SALT is not a flat cap.** It is the larger figure, phased down above a MAGI threshold and floored
  — a high earner is held to the floor by the phase-out, not capped at it.
- **Surtaxes ride on top.** Net investment income tax (3.8%) and the additional Medicare tax (0.9%)
  carry their own thresholds and belong in the total as separate components, not inside the bracket tax.
- **State — read `structure` first.** `GRADUATED` is a marginal ladder, `FLAT` is one rate from the
  first dollar, `NONE` is no state income tax, `NARROW_BASE` taxes only certain income (name it).
  Only SINGLE and MARRIED_FILING_JOINTLY columns exist; local income taxes are excluded; the
  thresholds run on the state's own taxable income, never federal.

## When anything is unavailable

Name what is missing, drop the analysis that depended on it, keep the rest. Never estimate, never
substitute a remembered value, never turn a missing value into zero.
