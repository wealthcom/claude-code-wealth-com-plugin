---
name: wealth-tax-insights
description: >-
  Runs a structured review pass over a client's filed tax return and returns the gaps and
  planning opportunities worth a client conversation — deductions and elections that are missing,
  figures that don't tie to each other, thresholds the household sits near, and what the return
  implies about the client's life. Surfaces what would otherwise take a careful line-by-line
  read, with the mechanism behind each item so the advisor can confirm it against the return.
  Use whenever an advisor asks what's worth looking at, flagging, or discussing: "anything we're
  missing on {client}'s return", "what should we be looking at for {client}", "planning
  opportunities for {client}", "review {client}'s 2025 return for gaps" — even if they never say
  "insights". Do NOT use for a descriptive overview of what the return says (use
  wealth-tax-executive-summary instead), or for projecting what a change would save (that is scenario
  modeling).
---

# Tax Insights

A structured review pass over a filed return that surfaces the items worth a client
conversation, so the advisor arrives at them directly rather than deriving them. Each item
carries its mechanism — the figure, where it comes from, and why it matters — so the advisor can
confirm it against the return quickly instead of rebuilding the analysis. Deliver inline as
Markdown. Match [references/example-riley.md](references/example-riley.md) for structure, tone,
and detail.

The executive summary describes the return; this surfaces what it implies and what it omits.
Overlap in figures is fine — what distinguishes an item here is that it is *anomalous, absent,
forward-looking, or inferential*, never a restatement of what the return plainly says.

## Critical

Seven rules. Each is expanded where it applies, but violating any one of them makes the output
unusable rather than imperfect.

1. **Every constant comes from a tool — `get_tax_constants` for federal, `get_state_tax_constants`
   for state.** Never state a threshold, cap, tier, rate, or bracket from memory. Federal figures
   re-index annually and recent statutory changes post-date training; state figures move more often
   still, with several states mid-phase-down.
2. **No directive language.** No "should", "recommend", "consider", "optimal", "best", or
   synonyms. State position and mechanism; the advisor decides what to do.
3. **Position, not projection.** Distances and rates are observational. What a change would save
   is scenario modeling and does not appear.
4. **Establish whose income it is before comparing to any ceiling.** Most per-person limits are
   per person, and household totals hide the attribution. See Compute.
5. **Marginal rate is computed from the return's position, never inferred from income size.**
   See Compute.
6. **No severity labels, urgency, or compliance-failure framing.** No "Critical", "urgent",
   "immediately", "failure to", "risk of penalty", and no instruction to file a specific form.
   An item that is wrong does damage in proportion to how alarming it sounded.
7. **A zero is a reading, not a default.** Before any item asserts that something is absent,
   missing, or zero, confirm the line was actually read and is actually empty. An unpopulated
   extraction field and a genuine zero are different facts, and treating them the same can produce false positives.

## Gather

Five calls. `get_lookback_report` is the spine — it now carries the household, the document
handles, and the rate reconciliation, so no separate baseline or document-listing hop is needed.

1. `find_client` → clientId/userId. Multiple matches: ask which.
2. `get_lookback_report` → the return. `taxYear` is optional; omit it and the tool probes the
   last four years itself. Four parts of the response matter here:
   - `household` → filing status, filers' DOBs, dependents. **Ages gate several checks** — IRMAA
     at 63, HSA at 65, QCD at 70½, RMD at 73 — so resolve them before detecting. Null DOBs are
     pruned from the response, so an age-gated check with no DOB is *unavailable*, not clean, and
     belongs under Limits rather than being silently skipped.
   - `returns[]` → one entry per return as `{ scope, jobId, vaultId, version }`, with `scope`
     being `federal` or `state`. This is how a year becomes a document for step 3.
   - `federal` and `state` → income by source, AGI, taxable income, `taxComposition`,
     `itemizedDeductions`, `bracketDistribution`, `businessProfile`, the Schedule C/D/E/F blocks,
     and the state summary with its effective and marginal rates.
   - `effectiveTaxRates` → see Compute. Use this rather than deriving a rate from `keyFigures`.
3. `get_extraction_form`, using the federal `jobId` and `vaultId` from `returns[]` → the
   line-level detail every Mode 1, 2, and 5 check depends on. The lookback's totals cannot support
   them. Payload is huge and saved to a file; search it. The labels each mode needs are listed per
   mode in the knowledge base.
4. `get_tax_constants` → the federal thresholds Mode 3 measures against. See
   [references/knowledge-base.md](references/knowledge-base.md) for the tool contract and what is
   not covered yet. Never supply a constant from memory.
5. `get_state_tax_constants` (tax year + state of residence) → the statutory state side: brackets,
   rates, standard deduction, exemptions. **Read `structure` first, because it decides which state
   checks exist at all** — `GRADUATED` is a rate ladder, `FLAT` is one rate from the first dollar
   with no band edge to sit near, `NONE` is a state that levies no income tax, and `NARROW_BASE`
   taxes only capital gains or interest and dividends. Pass `states`; omitted, it returns a
   one-line summary of every jurisdiction rather than brackets. What the household actually paid
   its state stays with the lookback — this tool is the law, not the return.

**The lookback response can be condensed under its character budget.** Bracket tables drop first,
then everything but `federal.keyFigures`; `effectiveTaxRates` and `returns` always survive, and
`note` says what happened. Read `note` before detecting — a condensed response with no
`bracketDistribution` cannot support Mode 3 bracket distances, and those belong under Limits
rather than being estimated from the rate summary.

## Detect

Six modes. Run all six before writing anything — items interact, and a life-event inference
changes how a threshold reads.

| Mode | The question | Lands in |
|---|---|---|
| 1 — Reconciliation | Does the return tie to itself? | Anomalies |
| 2 — Expected-but-absent | Is something missing that should be there? | Gaps, Triggers |
| 3 — Threshold proximity | Where does the household sit against a cliff or band edge? | Triggers |
| 4 — Composition ratio | What does the mix reveal that the totals don't? | Triggers, Strategy |
| 5 — Cross-field consistency | Do two fields that should agree, agree? | Anomalies |
| 7 — Life-event inference | What does this line imply about their life? | Gaps & Follow-Up |

Mode 6 (year-over-year delta) is deliberately held out of this version.

Each mode has a different false-positive profile and a different destination.

**Run the modes against a fixed walk order, not against the return freeform.** The knowledge base
carries a fourteen-stop coverage spine — a return review checklist's twelve sections, plus a payments
and withholding stop and a post-OBBBA stop — with the modes and specific
checks that apply at each. Walk the stops in order. Coverage does not emerge from good checks; a
review that finds four excellent items and never opened Schedule E is not a review.

[references/knowledge-base.md](references/knowledge-base.md) carries the spine, the named checks
under each mode, the suppression partner each needs, and which extraction labels each reads.

Most of the checklist is not a check. Triage every prompt: answerable from the return is a check;
answerable only with data the return doesn't carry is a **question in Gaps & Follow-Up**; advice
and needs-analysis prompts are out of scope.

Mode 2 is the highest-value mode and where the "surface it without the advisor digging" value
lives. Mode 1 is the most likely to embarrass — its checks are specific enough to be plainly
wrong when they misfire.

## Compute

Only what a check requires. The server computes most figures — take them as-is.

- **Effective rates come from `effectiveTaxRates.rates`, not from a ratio you build.** Each entry
  names its label, tax amount, income basis, and income amount, and the state-return-summary rate
  is labeled canonical. Where `basisIdentified` is `false`, **quote the dollars and not the
  percentage** — a rate whose denominator the tool could not identify is not a rate you can put
  in front of an advisor.
- **Establish whose income it is before comparing to any ceiling.** Most per-person limits are
  genuinely per person — the Social Security wage base, the Social Security component of
  self-employment tax, the excess-withholding credit, retirement account contribution limits. The
  lookback's household totals hide the attribution, and a check run against them will look
  arithmetically sound while being wrong. Additional Medicare Tax is the exception that
  aggregates. See the knowledge base for the full table.
- **Marginal rate is computed, never inferred from income magnitude.** This is a hard rule. A
  high-income return can sit at a low marginal rate through NOL carryforwards, suspended passive
  losses, or credit inventory, and an item written on the assumption that big income means a top
  bracket will be wrong in exactly the cases where it matters most. Read the rate off the
  return's actual position — `federal.rateSummary` and `state.stateMarginalRate`, against
  `bracketDistribution` — including carryforwards, before any item depends on it. On the state
  side, check `structure` before calling anything marginal: a flat state applies one rate from the
  first dollar and has no band to sit near, and a state with no income tax has no rate at all.
- **Each MAGI separately.** IRMAA MAGI is AGI plus tax-exempt interest; NIIT MAGI is AGI adjusted
  for foreign earned income; phase-outs use other definitions again. Computing MAGI once and
  reusing it misfires on exactly the households with large tax-exempt interest, where the shadow
  taxes bind.
- **Ordinary taxable income** = taxable income less preferential income. Bracket distances are
  measured on that remainder, never on total taxable income.
- **Distance to a threshold** is stated as a positive number with its direction in words. Never a
  signed figure.

Distance and rate are position. **Projected savings, and the dollar effect of a change the
household did not make do not appear.**

## Suppress

A pass, run after detection and before writing anything.

**Every item passes four gates before it renders.** Run these in order and drop
whatever fails:

1. **Read gate.** Was the figure this item depends on actually read from the return, and does it
   say what the item claims? An item asserting absence must point at a line that was read and is
   empty, not at a field that came back unpopulated.
2. **Age and eligibility gate.** Does every precondition hold for *this* household — filers'
   ages, enrollment status, plan coverage, entity type, filing status? An IRMAA item needs someone
   approaching Medicare. An RMD item needs someone past the RMD age *for their birth year*.
3. **Currency gate.** Is every rule and figure current for this tax year, from
   `get_tax_constants` or `get_state_tax_constants`? A correct calculation under a superseded rule
   is still wrong, and state rules turn over faster than federal ones.
4. **Consistency gate.** Does this item agree with the return and with every other item? An item
   computing on one cap while citing a different cap elsewhere in the same output is
   self-contradicting, and the advisor will see it before you do.

The gates decide whether an item may render; they do not catch the specific false positives that
recur. Those are the suppression rules in
[references/knowledge-base.md](references/knowledge-base.md) — read them before writing items.

## Presentation

- Markdown with headings; one table for the trigger positions. Items are prose, not bare bullets
  — each needs its mechanism, and a bullet invites an assertion without one.
- **State position and mechanism, never direction.** No "should", "recommend", "consider",
  "optimal", "best", or synonyms. Write what is true and why it matters; the advisor decides what
  to do.
- **Gaps & Follow-Up items are phrased as questions.** They are inferences, not findings, and the
  question form is what keeps an inference from reading as an assertion about the household.
- **Anomalies name the line.** "The 1099-R reports $48,000 gross with no taxable amount and no
  rollover notation" — not "there may be a reporting issue".
- Absolute figures only. If unverifiable, say so plainly and drop the item that depended on it.
- Label every figure. Pair every rate with its dollars and income basis, and name the provision
  beside any MAGI — "IRMAA MAGI", "NIIT MAGI" — since they are different numbers for the same
  household.
- One final figure per statement. Do the arithmetic before writing, not in the sentence.
- Attribute to the document ("the federal return shows"), never to a tool, field, or screen.
- Every figure comes from this client's return. Never carry an amount, rate, or threshold across
  from the reference example — its household is fictional.
- Cite constants from `get_tax_constants` and `get_state_tax_constants` on the Sources line.

## Structure

Title line: `# Tax Insights — {Family} Family — Tax Year {YYYY}`

Then these `##` sections, in this order. Omit a section with nothing in it rather than padding
it, except sections 5 and 6, which always appear.

| # | Section | Fed by | Rendering |
|---|---|---|---|
| 1 | `## Anomalies` | Modes 1, 5 | Where the return doesn't tie to itself. One `###` per item: the line, what it shows, what it should show, and what follows. These come first because they may be errors |
| 2 | `## Gaps & Follow-Up` | Modes 2, 7, and any Mode 5 check needing records the return doesn't carry | Questions for the advisor to take to the client. Each a question, then one line on what the return shows that prompts it |
| 3 | `## Triggers` | Modes 2, 3, 4 | Lead with the shadow-tax position, grouped by its shared driver rather than by a shared figure — each threshold measures its own MAGI. Then a **table**, `Threshold \| This household \| Distance`, covering every threshold measured including those the household is far from; the household column names the measure it used ("$800,238 IRMAA MAGI"), never a bare amount. Then a short `###` per trigger that carries a consequence |
| 4 | `## Strategy signals` | Mode 4 | What the mix reveals. Prose, one paragraph per signal. Composition only — no modeled outcome |
| 5 | `## Checked, nothing observed` | All modes | Bulleted, each with a one-line reason. This is what makes silence mean "checked" rather than "missed" |
| 6 | `## Limits of this review` | — | Two or three lines: that it is observational rather than modeled, what the review read, and anything it needed and could not get |

**Shadow taxes** is the advisor-facing name for the AGI/MAGI-driven cliffs — net investment
income tax, IRMAA, and the phase-outs. Use it as the lead concept in Triggers; it is legible in a
way that naming each surtax separately is not. It groups them by cause, not by figure: each
provision modifies AGI its own way, so the lead never states a single household MAGI, and every
MAGI in the output carries the name of the provision it belongs to.

Section 5 reports **against the coverage spine**, not against
whatever happened to get examined. That is the only thing that makes them worth reading: an
advisor cannot act on items without knowing what was looked at, and a list of stops with no
defined universe behind it is a claim the skill can't back. Both are nearly free — the walk
already visited every stop.

[references/example-riley.md](references/example-riley.md) is this spec already rendered — match
it for tone and level of detail.

## Common Issues

**The lookback returns a different tax year than the advisor named.** With `taxYear` omitted the
tool probes the last four years and returns the most recent it finds. Pass `taxYear` whenever the
advisor names one. If the response comes back with a different year, say so in the first line of
the output and confirm before continuing — reviewing the wrong year silently is worse than
returning nothing.

**`note` says the response was condensed.** Bracket tables drop first, then everything but
`federal.keyFigures`. Shadow-tax positions still work, since they run off AGI and MAGI. Band-edge
distances do not — put them under Limits rather than reconstructing them from the rate summary.

**A filer's DOB is missing.** Null values are pruned, so an absent DOB is indistinguishable from
a household where age is irrelevant. Age-gated checks — IRMAA at 63, HSA at 65, QCD at 70½, RMD at
73 — become *unavailable*, not clean. They go under Limits, never under "Checked, nothing
observed."

**`get_tax_constants` has no data for the tax year.** The response carries `error`,
`taxYearsCovered`, `searchInstead`, and `sourcePrecedence` instead of `constants`. Search the
named publishers and say which source you used. If a figure still can't be verified, drop the item
that depended on it and record it under Limits. Never estimate, never fall back on a remembered
value.

**`get_state_tax_constants` has no data for the tax year, or returns `detailAvailable:
"STRUCTURE_ONLY"`.** Structure-only carries whether the state levies an income tax and how, with no
rates or thresholds — do not borrow either from an adjacent year. The structure checks still run;
state band-edge distances go under Limits. Items resting on the state rate the lookback already
computed are unaffected.

**A rate carries `basisIdentified: false`.** Quote the dollar amount and not the percentage. A
rate whose denominator the tool could not identify is not a rate to put in front of an advisor.

**The extraction form is too large to read.** It is saved to a file — search it for the specific
labels each mode needs rather than reading it through. The per-mode label lists are in
[references/knowledge-base.md](references/knowledge-base.md).

**`find_client` returns more than one match.** Ask which client rather than picking. Two households
with similar names produce a review that is entirely correct about the wrong person.
