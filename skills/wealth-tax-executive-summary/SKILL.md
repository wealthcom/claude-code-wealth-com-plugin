---
name: wealth-tax-executive-summary
description: "Executive summary" of a client's filed tax return for a financial advisor — a descriptive tax overview (household, income, deductions, tax & rates, capital-gains and IRMAA positioning, payments & safe harbor, schedules). Use whenever an advisor wants to understand a client's current tax situation — "executive summary / tax overview for {client}", "summarize {client}'s 2025 return", "what's {client}'s tax situation" — even if they don't say "executive summary". Descriptive only; excludes insights/opportunities and scenario modeling (that's wealth-tax-scenarios). Do NOT use for estate documents — a will, trust, or power of attorney gets the estate document summary instead (wealth-estate-document-deep-dive), even when the advisor asks for an "executive summary" of one.
---

# Tax Executive Summary

A descriptive read of one filed tax year: what the household filed, what it shows, and where it sits against the tax law for that year. Nothing modeled, nothing recommended — that is `wealth-tax-scenarios`' territory, reached only by explicit handoff.

## Critical

Read this before anything else.

- **No tool fires until the client is resolved.** Name the client and the tax year in your first sentence.
- **Descriptive only.** Never model a change, never suggest a strategy, never say what they "should" do. An advisor asking "what if" or "what would it cost to..." is asking for `wealth-tax-scenarios` — hand off explicitly, don't reach for a calculation tool here.
- **This skill reads filed years. `wealth-tax-scenarios` builds unfiled ones.** If the household hasn't filed the year the advisor names, say so plainly and offer to build an estimate instead — don't invent one here.
- **Never a bare rate.** Every percentage is paired with the tax dollars and the income amount that produced it, in the same sentence. If no income figure in the return reproduces a rate, give the dollars and drop the percentage — never estimate a denominator.
- **No blended federal+state marginal rate.** Federal and state marginal rates are never added into one "your marginal rate is X%" figure — keep them separate. A combined federal+state figure is fine, but only in dollars (total tax), never as a summed rate.
- **Effective rates come from the return's own rate summary, taken whole.** Never recompute, average, or reconcile an effective rate yourself.
- **The ordinary marginal bracket is computed on ordinary income only.** Subtract preferential income (long-term capital gains, qualified dividends) from taxable income *before* finding the bracket. The bracket that total taxable income falls into is usually the wrong answer — it's a common, quiet error.
- **Do the arithmetic before writing, not in the sentence.** One final figure per statement. No "35%... actually 24%" visible correction.
- **Absolute figures only.** No "about," "roughly," or "~" on anything the return states exactly. If a figure truly can't be pinned down, say so plainly instead of hedging.
- **Attribute every figure to the document it came from** — "the 2025 federal return shows," "the state return reports" — never to a screen, tab, page, or field name. Never name a tool.
- **Every figure comes from this client's return.** Never carry a number, rate, bracket, or state across from a reference example or another client — the fixture data is fictional and wrong here.
- **Estate documents route elsewhere.** A will, trust, or POA gets `wealth-estate-document-deep-dive`'s summary, never this one, even if the advisor says "executive summary."
- **Omit a section only if its data is genuinely unavailable — and say so in its place.** Never silently drop one of the eight.

## Quick Start

1. **Resolve the client**: `find_client`.
2. **Pick the year**: the one the advisor named, or the most recent filed year if they didn't say.
3. **Pull the filed-year report**: `get_lookback_report` — this is the primary source for seven of the eight sections.
4. **Pull tax law context**: `get_federal_tax_constants` (IRMAA tiers, capital-gains breakpoints) and `get_state_tax_constants` if a state return exists.
5. **Only reach for line-level detail** — `get_extraction_form` — when the lookback report doesn't carry something a section needs (commonly: the "what's in the return" schedule list, or a specific safe-harbor line).
6. **Build all eight sections in fixed order.** Markdown headings and tables — never continuous prose. A wall of paragraphs is the most common way this goes wrong.

## Workflow

### Step 1 — Resolve the client and the year

```
find_client with what the advisor said
- one match          -> proceed, name the household and the year in your first sentence
- advisorMustChoose   -> ask which household. Never an id to disambiguate
- zero                -> say so, don't fall back to another client

get_lookback_report(wid, taxYear?)
- no taxYear given by advisor -> omit it; the tool returns the most recent filed year with a report
- no report exists for the named year -> say plainly no filed return is on file for that year;
  offer wealth-tax-scenarios to build an estimate instead. Stop here.
```

### Step 2 — Pull tax law context

```
get_federal_tax_constants([taxYear]) — capital-gains breakpoints, IRMAA MAGI tiers, AMT/NIIT
  thresholds for the filed year
get_state_tax_constants([stateCode], [taxYear]) — only if the household filed a state return;
  tells you whether that state is GRADUATED / FLAT / NONE / NARROW_BASE, which decides how
  Section 4's state rate line reads
```

### Step 3 — Build the eight sections, in this fixed order

Title line first: `# Tax Executive Summary — {Family} Family — Tax Year {YYYY}`

1. **Household & filing** — filers, filing status, state of residence, dependents.
2. **Income** (table) — every income line the report carries, **bold Total row**.
3. **Deductions** (prose) — standard vs. itemized, what's claimed, the SALT cap if it binds.
4. **Tax and rates** (table, then prose) —
   - Table: **bold Total federal tax row**, plus state tax if applicable.
   - Then, as separate lines: federal effective rate (with dollars + income basis), federal marginal bracket (computed on ordinary income only, per Critical), state effective rate if the state has one (structure-dependent — see `reference/tax-computation-rules.md`), state marginal rate if applicable.
   - Then: the combined federal + state tax **in dollars**, never as a summed rate.
5. **Capital gains bracket positioning** — where net long-term gains sit against that year's breakpoints (from `get_federal_tax_constants`), in dollars and bracket name, not a bare percentage.
6. **IRMAA positioning** (table by MAGI tier) — this year's MAGI against the tier breakpoints; state which tier they're in and the distance to the next one, in dollars.
7. **Payments and safe harbor** — withholding, estimated payments, and whether safe harbor was met against last year's liability (110% for higher-income filers) or this year's.
8. **What's in the return** (schedule table) — every form/schedule the report or `get_extraction_form` lists, one row each.

Omit a section only if the report has nothing for it, and say so in that section's place rather than skipping it silently — e.g., "No capital gains reported this year, so there's no bracket position to show."

### Step 4 — Stop with one offer

End with exactly one next step, phrased as a question — e.g. offer the full line-item return, or hand off toward `wealth-tax-scenarios` if the advisor's next question is forward-looking. Never a menu of options.

## Answer shapes

**Full summary**: title line → eight sections in fixed order, each a heading + table/prose → omissions stated in place, not skipped → one offer
**No filed return for the named year**: say so plainly → offer to build an estimate via `wealth-tax-scenarios` → stop
**Estate document named ("summarize the trust")**: redirect to `wealth-estate-document-deep-dive` — don't produce a tax summary of a non-tax document
**Forward-looking question mid-summary** ("what if they convert some of that IRA"): hand off explicitly to `wealth-tax-scenarios` — don't reach for a calculation tool here
**Rate with no reproducible basis**: dollars only, percentage omitted, say why

## Common Issues

**"Give me the executive summary for the Bradys' trust"**: that's an estate document, not a tax return — route to `wealth-estate-document-deep-dive` even though the advisor used the words "executive summary."
**"What would they owe if..."**: descriptive only here. Hand off to `wealth-tax-scenarios` rather than modeling anything.
**Two different rates appear for the same concept**: give both, in dollars-and-basis form, say which one the advisor sees on the Tax Planning screen, and which you're using here.
**No income figure reproduces a stated rate**: give the dollars, drop the percentage, say plainly why.
**Household hasn't filed the year asked about**: say so, offer `wealth-tax-scenarios` instead of guessing or substituting the nearest filed year.
**Advisor asks for "just the total tax"**: still give the combined figure attributed to its components — never a bare number with no source.

## Examples

- [examples/filed-return-review.md](examples/filed-return-review.md) — full session on the Riley household: the eight-section summary, a rate with no reproducible basis, and a forward-looking question handed off to `wealth-tax-scenarios`.

## Reference

- [reference/tax-computation-rules.md](reference/tax-computation-rules.md) — the effective-rate, marginal-bracket, and state-structure rules in full, with worked examples.
