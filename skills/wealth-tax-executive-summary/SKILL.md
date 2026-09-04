---
name: wealth-tax-executive-summary
description: >-
  "Executive summary" of a client's filed tax return for a financial advisor — a descriptive tax overview (household, income, deductions, federal tax & rates,
  state tax, capital-gains and IRMAA positioning, payments & safe harbor, schedules). Use whenever an advisor wants to understand a client's current tax
  situation: "executive summary / tax overview for {client}", "summarize {client}'s 2025 return", "what's {client}'s tax situation" — even if they don't
  say "executive summary". Descriptive only; excludes insights/opportunities and scenario modeling. Do NOT use for estate documents — a will, trust, or power of
  attorney gets the estate document summary instead, even when the advisor asks for an "executive summary" of one.
---

# Tax Executive Summary

A descriptive read of a client's filed return, so the advisor can brief themselves and the client before any strategy is layered on top. Delivered inline as Markdown.

The server computes most of the figures. This skill reads them, verifies the tax-law thresholds against the constants tools, and renders them in a fixed shape.

Strictly descriptive — no recommendations, no gaps, no "worth flagging." Match [references/example-riley.md] for structure, tone, and detail.

## Critical

Read this before anything else. Each line ends with the rule it protects.

- Descriptive only. No recommendations, no opportunities, no gap analysis — that is a different skill.
- Never state a tax-law figure from memory. Brackets, caps, thresholds, tiers, and percentages come from the constants tools; they re-index yearly and recent changes post-date training.
- Every figure comes from THIS client's return. Never carry a number, rate, bracket, or state across from the reference example — its household is fictional.
- Never attribute an income item to a spouse unless the client record itself attributes it to that filer. Wages on line 1z — and any other household total — are combined figures that name no one; render them unnamed ("Wages (W-2)"), and name a filer only when the source does. Being the primary filer, or being listed first in a schedule header, is not attribution. This protects against inventing a filer attribution the return does not support.
- Absolute figures only — no "about/roughly/~". Label every figure ("total federal tax of $X", not a bare "$X").
- Never give a bare effective rate. Every effective rate — federal and state alike — is total tax ÷ AGI, and you always show that division (e.g. "federal effective rate 24.2%: total federal tax $191,615 ÷ AGI $791,038"; "state effective rate 7.4%: total state tax $58,204 ÷ AGI $791,038"). AGI is always the denominator — never taxable income, never state taxable income. Pair a marginal rate with the band it applies to, and never blend federal and state into one marginal rate.
- One final figure per statement. Do the arithmetic before writing — no visible corrections, no "actually," no "35%… no, 24%."
- IRMAA is a two-year lookback. Always state which premium year this return sets, and which year's tiers you measured against — name the proxy year when the real one isn't published yet.
- Bracket-positioning tables (§5) carry exactly one `◀` marker each — the band the household occupies. Ladder rows come from the constants tools, never memory.
- Estate documents do not belong here. A will, trust, or POA gets the estate document summary; stop and offer that instead.

## Quick Start

1. `find_client` → clientId. Multiple matches: ask which.
2. `get_lookback_report` → income, AGI, deductions, tax composition, rates, brackets, and the return's `jobId`/`vaultId`.
3. `get_extraction_form` (federal `jobId` + `vaultId`) → the payment lines the lookback omits.
4. `get_federal_tax_constants` → federal thresholds (standard deduction, SALT, capital gains, brackets, surtaxes, safe harbor, IRMAA).
5. `get_state_tax_constants` → the state's statutory brackets and structure.
6. Compute ages, rates, positioning. Render the nine sections in Structure.

## Workflow

### Step 1 — Resolve the client

- `find_client` with what the advisor said → clientId/userId.
- Several matches → ask which, listing names. Never pick silently.
- Zero matches → say so. Never summarize the nearest name.

### Step 2 — Read the lookback report

`get_lookback_report` (only `clientId` required) returns almost everything in one call:

- `federal` / `state` — income by source, AGI, taxable income, deductions, federal tax composition (income tax, SE, additional Medicare, NIIT), federal and state brackets, state summary, federal safe-harbor figure.
- `household` — filing status and each filer's date of birth.
- `effectiveTaxRates` — every rate in the record, each with its tax amount and income basis.
- `returns` — the `jobId` + `vaultId` of each return, tagged `scope: "federal"` or `scope: "state"`. Step 3's federal job comes from here.

Take the tax year from `taxYear` on the report, never from a document name. Pass `taxYear` only when the advisor named one — omitted, the tool picks the most recent filed year. If `returns` has no federal entry, fall back to `list_documents` and use the `lookbackEligible: true` row.

### Step 3 — Pull the payment lines from the return

`get_extraction_form` (federal `jobId` + `vaultId`) → payment lines not in the lookback. The payload is large and saved to a file; search it for these labels only:

- Line 25d (withholding; 25a/25c for the W-2 vs additional-Medicare split)
- Line 26 (estimated payments)
- Line 37 (balance due), Line 38 (penalty)
- Schedule D Line 13 (capital gain distribution)

### Step 4 — Get the federal constants

`get_federal_tax_constants({ taxYear, filingStatus, sections })` → SALT cap/phase-out, standard deduction + age-65 addition, LTCG thresholds, NIIT/Medicare thresholds, IRMAA tiers, safe-harbor percentages.

The normal call requests `STANDARD_DEDUCTION`, `SALT_CAP`, `CAPITAL_GAINS`, `ORDINARY_BRACKETS`, `SURTAXES`, `SAFE_HARBOR`, and `IRMAA`. See [references/knowledge-base.md](references/knowledge-base.md) for the full contract and graceful degradation. Never supply these from memory.

### Step 5 — Get the state constants

`get_state_tax_constants({ taxYear, states, filingStatus })` → the state's statutory brackets, rates, standard deduction, and exemptions.

The lookback's state brackets are the ones this return was filed against and stay primary; this tool supplies the thresholds you quote and covers a condensed or absent lookback. Read `structure` first: `GRADUATED` is a rate ladder, `FLAT` is one rate from the first dollar, `NONE` means no state income tax, `NARROW_BASE` taxes only capital gains or interest-and-dividends. Only `SINGLE` and `MARRIED_FILING_JOINTLY` columns exist, and local income taxes are excluded. See [references/knowledge-base.md](references/knowledge-base.md) for its limits and precedence.

### Step 6 — Compute

- **Ages** = tax year − birth year, from `household` (65+ if reached 65 by year-end). Show age only, never DOB. A filer with no DOB gets no age — omit it, and claim no age-65 addition you can't support.
- **Income attribution** — attach an income item to a specific spouse ONLY when the return or client record attributes it to that filer. A combined household line — 1040 line 1z wages is the common one — names no one; render it without a filer name. Do not infer the earner from who is the primary filer, whose name a schedule header lists first, or which spouse "usually" earns it. When in doubt, leave it unnamed.
- **Age-adjusted standard deduction** = base + age-65 addition per qualifying filer (the lookback's stored figure omits the addition — use yours).
- **Effective rate = total tax ÷ AGI, always.** Compute both the federal effective rate (total federal tax ÷ AGI) and the state effective rate (total state tax ÷ AGI) yourself from those two figures. Do not read a pre-computed rate off `effectiveTaxRates.rates`, and never use taxable income or state taxable income as the denominator even when the record offers one. If AGI or the total-tax figure is unavailable, give the tax in dollars and leave the percentage out.
- **Ordinary marginal bracket** = subtract preferential income (LT gains + qualified dividends) from taxable income first, then find that remainder in the bracket table. The bracket that *total* taxable income falls into is the usual wrong answer.
- **Income-tax build (for §4)** — income tax = the ordinary-bracket tax on (taxable income − preferential income) + the preferential-rate tax on the preferential slice. Show the two parts and their sum; they should reconcile to the income-tax figure on the return.
- **State rate** depends on structure (Step 5). Graduated → a marginal bracket, read off *state* taxable income, never federal. Flat → the flat rate, no marginal sentence. None → say so in place of a rate. Narrow-base → say what the rate reaches, not "the household's state rate." Where a local income tax applies, add that the state rate is not the whole state-and-local liability.
- **Capital-gains position** — preferential income stacks above ordinary taxable income; map that band onto the 0/15/20% thresholds; NIIT 3.8% stacks where it applies.
- **Net long-term gain** = (proceeds − basis) + Schedule D Line 13, reading Line 13 directly.
- **IRMAA** — MAGI = AGI + tax-exempt interest. This return sets premiums for **premium year = tax year + 2** (the two-year lookback). `get_tax_constants` resolves it and returns `premiumYear`, `published`, and `tiersShownForPremiumYear`:
  - `published: true` → measure MAGI against the premium year's own tiers.
  - `published: false` → the premium year isn't published yet. Measure against the most recent published year's tiers as a proxy (the tool returns them), and say plainly which year you used and that the real thresholds may shift. A 2025 return sets 2027 premiums; if 2027 isn't published, measure against the 2026 (or 2025) tiers and say so.
  - Never present the filing year's own tiers as the household's future tier without naming the proxy.
- **Safe harbor / penalty** — no federal penalty if withholding met a safe harbor (90% current-year, or 110% prior-year when AGI > $150k). The lookback's safe-harbor dollar figure is forward-looking (next year's estimated-payment target), not a balance owed.

## When a step comes back short

Name what is unavailable, drop only the analysis that depended on it, and deliver the rest. Never substitute a figure from memory or from the reference example.

- **`find_client` returns several people** — ask which, listing names.
- **`get_lookback_report` says no filed return** — ask which year, or say none is on file. Do not build from `get_extraction_form` alone.
- **It says the document is not a tax return** — the advisor named an estate document. Stop; offer the estate document summary.
- **`note` says the view is condensed** — bracket tables were dropped. Use `get_tax_constants` / `get_state_tax_constants` for brackets, and say the bracket detail came from the published tables.
- **AGI or the total-tax figure unavailable** — give the tax in dollars and leave the effective percentage out. Do not fall back to taxable income as the denominator, and do not mention that a basis is missing. This will confuse the user.
- **income the return does not attribute to a filer** — report it unnamed. Never guess the spouse; if the advisor asks whose it is, say plainly it isn't attributed on the return and point to the underlying W-2 / source form.
- **`get_extraction_form` has no reviewable return** — say the payments and safe-harbor detail cannot be confirmed, and keep Section 8 short rather than omitting it.
- **`get_federal_tax_constants` does not cover the year** — follow [references/knowledge-base.md](references/knowledge-base.md): search the publishers it names, cite the source, never fill from memory.
- **`get_state_tax_constants` uncovered, or `detailAvailable: "STRUCTURE_ONLY"`** — give the state tax in dollars from the lookback, leave the marginal state bracket out, and say the state bracket detail could not be verified. Borrow no thresholds from an adjacent year.

## Presentation

- Markdown with headings and tables, never continuous prose. A wall of paragraphs is how this summary most often goes wrong.
- Write for an advisor about to walk into a review: plain, warm, and readable. Tables carry the numbers; short prose connects them. Readable is not the same as chatty — still no prose walls.
- Descriptive throughout. No "talking points," no "worth flagging / worth confirming," no bunching / PTET / harvesting / entity suggestions, no "they may not realize" framing. Naming what is on the return (e.g. a surtax subtotal, a floored SALT deduction) is descriptive; suggesting what to do about it is not — that is the insights and scenario skills.
- Attribute figures to their document ("the federal return shows," "the state return reports") or to the return's own line/schedule in a `Source` column. Never cite where a figure is displayed — no screen, page, or tab names. Never name tools, fields, or process. The client record may itself label a figure as "the figure the Tax Planning screen shows" (or carry a flag to that effect); that label is internal — strip it. Never write "the Tax Planning screen shows / says …" or attribute any figure to a screen; if two figures for the same concept disagree, distinguish them by their arithmetic (the tax dollars and the income basis behind each), not by where one is displayed.
- Attribute income to a filer only when the source does (see the Critical rule and Step 6). A combined line — wages on 1040 line 1z above all — is rendered without a name; do not append "(David)" or "(Gregory)" to it on a hunch.
- Combined fed+state figure in dollars only — never a blended marginal rate.
- Omit a section only if its data is truly unavailable, and say so in its place.

## Structure

Render as Markdown. Five sections carry a table (Bracket positioning carries two) — they are what make the summary scannable, and they are not optional.

Title: `# Tax Executive Summary — {Family} Family — Tax Year {YYYY}`

Then these eight `##` sections, in order:

| # | Section | Rendering |
|---|---|---|
| 1 | `## Household & filing` | One line: filing status and state of residence. Then a bullet per filer with age at year-end. Attach an income type to a filer (e.g. "Wage income (W-2)") only if the record attributes it to that person; otherwise describe the household's income sources without pinning them to a spouse |
| 2 | `## Income` | A **waterfall table** flowing from income sources to taxable income: `Step \| Amount \| Source`. One row per income source, then a bold **Total income** row, then above-the-line adjustments (e.g. ½ SE tax) as negative rows, a bold **Adjusted gross income** row, the itemized/standard deduction as a negative row, and a bold **Taxable income** row. The `Source` column names the return's own line or schedule (never a platform screen) — omit an entry you can't verify. Name a filer in a row label (e.g. "Wages (W-2, David)") only when the source attributes that income to that filer; a combined line like 1040 line 1z gets no name. Then tax-exempt interest as a trailing line |
| 3 | `## Deductions` | Prose. Itemized composition; the SALT position (the pre-cap state-and-local total against the capped/floored amount); the age-adjusted standard deduction for comparison; QBI. The waterfall (§2) already carried the arithmetic to taxable income, so this section explains it rather than re-deriving it |
| 4 | `## Federal tax and rates` | Lead line splitting taxable income into its ordinary and preferential parts, then total federal tax. **Table** `Component \| Amount` — the income-tax row shows the ordinary + preferential build — closing with a bold **Total federal tax** row. Then a compact rates block: federal effective (total federal tax ÷ AGI, shown as that division), federal marginal on ordinary income (plus the 0.9% additional-Medicare note), and federal marginal on preferential income (plus NIIT). Keep marginal ordinary and preferential separate; positioning detail is in §5 |
| 5 | `## Bracket positioning` | Two **marker tables**, ordinary income then long-term capital gains / qualified dividends — each a bracket ladder for the filing status with one `◀` marker on the band the household occupies. Same idiom as the IRMAA table in §7. See "Marker tables" below |
| 6 | `## State tax` | Brief. State income tax in dollars; effective state rate (total state tax ÷ AGI, shown as that division) and (if graduated) marginal state rate — or a line saying the state has no income tax / a narrow base; the state withholding and balance owed; then combined federal + state income tax in dollars. Add the local-income-tax caveat where it applies |
| 7 | `## IRMAA positioning (Medicare premium surcharge)` | This is only applicable if at least one spouse is 63 or older. Lead line with MAGI, the premium year this return sets, and the tier year measured against — naming the proxy when the premium year is unpublished. **Table** `MAGI ({status}) \| Part B / month \| Part D add'l / month`, every tier a row. Then where this household lands and what it costs per enrolled spouse |
| 8 | `## Payments and safe harbor` | Prose, federal only. Withholding, estimated payments, balance owed, and whether a safe harbor was met |

### Marker tables (§5)

Both §5 tables are bracket ladders with a position marker — the same "ladder plus where the
household lands" shape as the §7 IRMAA table. Three columns: the rate, the taxable-income range for
the filing status, and a blank-headed third column that holds the `◀` marker on the one occupied
row.

- **Rows come from the constants tools, never memory.** Ordinary bands from `get_federal_tax_constants`
  `ORDINARY_BRACKETS`; the 0/15/20 bands from `CAPITAL_GAINS`. Ranges are the statutory thresholds
  for the filing status, stated in absolute dollars.
- **Exactly one `◀` marker per table**, on the band the household occupies, and bold that row. Never
  mark two rows.
- **Ordinary table** — the marker sits on the band containing *ordinary* taxable income (total
  taxable income less preferential income, per Compute). The marker text names that figure and the
  dollars to the next band up.
- **Capital-gains table** — preferential income (LT gains + qualified dividends) stacks above
  ordinary income, so the slice occupies the range from ordinary taxable income up to total taxable
  income. The marker sits on the band that range lands in, and its text names the slice in dollars,
  the stacking range, and — where it applies — that NIIT (3.8%) stacks on top.
- **When the bands can't be verified** (condensed lookback and an uncovered constants year), drop
  the affected ladder and state the marginal bracket in words instead, per "When a step comes back
  short." Never render a ladder from remembered thresholds.

[references/example-riley.md] is this spec rendered — match it for tone and detail. Note that its income rows attribute wages to David and the Schedule C business to Victoria only because that fictional example's source data attributes them to those filers. Do NOT copy that filer attribution as a pattern: apply it to a real client only when the client's own record attributes the income to that spouse (see the Critical rule and Step 6).
