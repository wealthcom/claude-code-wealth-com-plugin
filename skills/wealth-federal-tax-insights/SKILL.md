---
name: wealth-federal-tax-insights
description: >-
  Runs two passes over what's available for a client — a return-integrity pass and a planning-
  observation pass — and returns what's worth a client conversation: figures that don't tie,
  deductions or elections that are missing, thresholds the household sits near, and the planning
  conversations the facts support. Adapt to what exists — a full return, a return plus supporting documents, documents
  alone, or facts the advisor supplies. Use whenever an advisor asks what's worth looking at,
  flagging, or discussing: "anything we're missing on {client}", "planning opportunities for
  {client}", "review {client}'s 2025 return for gaps", "what should we be talking to {client}
  about" — even if they never say "insights". Do NOT use for a descriptive overview of what the
  return says (that is wealth-tax-executive-summary), or for projecting what a change would save (that is
  Wealth-tax-scenarios).
---

# Tax Insights

Two passes over the client's available materials. The **integrity pass** asks whether what's
filed ties to itself and whether anything expected is absent. The **planning pass** asks which
client conversations the facts support. Both carry equal weight — the integrity pass keeps the
accuracy controls, the planning pass gives advisors well-supported reasons to start a
conversation. Every item is a quick hitter: enough of its basis and the provenance of each fact
for the advisor to judge whether it applies and act on it — no more, because the advisor digs
deeper by asking. Deliver inline as Markdown; match
[references/example.md](references/example.md) for structure, tone, and detail.

The executive summary describes the return; this surfaces what it implies, omits, and makes
possible. Never restate what the return plainly says — an item here is *anomalous, absent,
forward-looking, or a supported planning conversation*. Open at the first section that has content —
no preamble, no household or income recap; that overview is the executive summary's job.

## Critical

The non-negotiables. Violating any one makes the output unusable rather than imperfect.

1. **Constants come from a tool — never memory.** `get_federal_tax_constants` for federal,
   `get_state_tax_constants` for state. Every threshold, cap, tier, rate, and bracket re-indexes,
   and recent statutory changes post-date training.
2. **No directive language.** No "should", "recommend", "consider", "optimal", "best", or
   synonyms. State position and mechanism; the advisor decides what to do.
3. **Observation, not projection.** Distances and rates are observational. What a change would
   save is scenario modeling — hand it off, never compute it here or invent a savings estimate.
4. **Every fact keeps its source.** Label each material fact *return-confirmed*,
   *supporting-document*, *advisor-provided*, or *unknown*. Never imply a return confirmed an
   advisor's assumption. When sources disagree, present both and ask rather than choosing silently.
5. **Entity type and attribution are read, never inferred.** Whose income it is (per-person
   limits) and whether a business is a Schedule C or a passthrough come from a source or a question —
   never from income size or occupation. The name on a form or schedule is read from
   `get_extraction_form`; the lookback carries none, so attribution rests on that call or a question.
6. **Marginal rate is computed from the return's position, never inferred from income size.**
7. **No severity, urgency, or compliance-failure framing**, and no instruction to file a specific
   form. A wrong item does damage in proportion to how alarming it sounded.
8. **Ground every item in a fact actually read.** An absence or zero fires only after the line was
   read and found empty — an unpopulated field is not a zero. A planning item fires only on facts
   specific to *this* client; high income or net worth alone never produces a generic catalogue.

## Starting points

The materials determine the depth. Let the depth show through the output's evidence — the facts
each item cites — not through process narration or a standing limitations notice.

- **Full return** — the ideal spine. Run the complete integrity pass plus the planning pass, with
  line-level evidence where a finding depends on a specific amount or omission.
- **Full return + supporting documents** — run the full review, then read only the documents that
  answer an active planning question (W-2s, paystubs, K-1s, brokerage statements, plan documents).
  Do not read every document indiscriminately.
- **Supporting documents, no return** — a partial planning scan. Documents can reveal
  compensation, withholding, ownership, distributions, gains, and account characteristics; they do
  not support a claim that the return reconciles or is complete. State that limit in the output.
- **Advisor-provided context** — facts in the conversation may start or refine the analysis.
  Preserve them as advisor-provided; ask follow-ups where a missing fact decides whether an
  opportunity applies.

The tooling gives line-level precision for a full return and less for general documents. Respect
the difference; do not treat every document reading as equally authoritative or complete.

## Gather

**Full-return path — five calls.** `get_lookback_report` is the spine; it carries the household,
the document handles, and the rate reconciliation, so no separate baseline hop is needed.

1. `find_client` → clientId/userId. Multiple matches: ask which.
2. `get_lookback_report` → the return. `taxYear` optional (omit and it probes the last four
   years). Four parts matter: **household** (filing status, DOBs, dependents — ages gate IRMAA at
   63, HSA at 65, QCD at 70½, RMD at 73; a null DOB makes an age-gated check *unavailable*, not
   clean); **returns[]** (`{scope, jobId, vaultId, version}`); **federal/state** (income, AGI,
   taxable income, `taxComposition`, `itemizedDeductions`, `bracketDistribution`,
   `businessProfile`, the Schedule blocks); **effectiveTaxRates** (use these, don't derive a
   rate). Read `note` — a condensed response may have dropped `bracketDistribution`.
3. `get_extraction_form` (federal `jobId` + `vaultId`) → the line-level detail the integrity
   checks depend on, and the source for **whose name is on each form and schedule** — the Schedule C
   proprietor, the Schedule SE filer per spouse, the owner of a 1099 — which the lookback does not
   carry and which decides whether a per-spouse insight applies. Pull it, and read the names, before
   attributing any schedule to a filer. Huge, saved to a file; search it for the labels each check
   needs (listed in the knowledge base).
4. `get_federal_tax_constants` (taxYear + filingStatus) → the federal thresholds. Never a constant from
   memory.
5. `get_state_tax_constants` (taxYear + state) → the statutory state side. Read `structure` first
   — `GRADUATED` / `FLAT` / `NONE` / `NARROW_BASE` decides which state checks exist at all.

**Documents-only or advisor-context path.** Resolve the client, then `list_documents` and
`get_extraction_result` for what's uploaded; take remaining facts from the conversation. The
constants tools still supply every rule. There is no reconciliation spine, so the output is a
planning scan — do not claim the return reconciles or is complete; where a missing record blocks an
insight that otherwise looks applicable, name it under Where more is needed.

See [references/knowledge-base.md](references/knowledge-base.md) for the tool contracts,
per-check extraction labels, and what the constants tools do not cover yet.

## Detect — two passes

Run both before writing anything. Items interact, and a planning inference changes how a threshold
reads.

**Integrity pass — does what's filed tie, and is anything expected absent?** Four check families,
run against the fixed fourteen-stop coverage spine in the knowledge base, not the return freeform.
Coverage is enumerated, not emergent: a review that finds four good items and never opened
Schedule E is not a review.

| Check | The question | Lands in |
|---|---|---|
| 1 — Reconciliation & consistency | Does the return tie to itself, and do fields that should agree, agree? Three layers — internal ties, inter-form presence, and the schedule carry map; see the KB | Return anomalies |
| 2 — Expected-but-absent | Is something missing that should be there? | Questions to validate, Threshold signals |
| 3 — Threshold proximity | Where does the household sit vs a cliff or band edge? | Threshold & timing signals |
| 4 — Composition ratio | What does the mix reveal that the totals don't? | Threshold signals, Planning opportunities |

**Planning pass — which client conversations do the facts support?** Synthesizes facts across
sources rather than reading one line in isolation. A trigger family renders **only when
client-specific facts activate it**. Families:

business ownership and entity structure; compensation, equity, and benefits; retirement-plan
design and contribution capacity; QBI eligibility and limitation positioning; PTET and other state
elections; investment taxation, gain/loss timing, and asset location; charitable planning;
real-estate ownership, depreciation, and elections; estimated payments and withholding; Medicare,
IRMAA, and retirement transitions; state residency and relocation; business succession and
liquidity events; and dependents, education, marriage, divorce, inheritance, and other life events.

The **conditional-observation rules** — NIIT exposure, backdoor-Roth eligibility,
mortgage-interest value, HSA contribution vs. limit, safe-harbor cushion — each fire only when the
client's own figures cross the specific threshold, never as a generic reminder. An always-fire
"reminder" is the anti-pattern to suppress. Named rules, triggers, and the constants each needs are
in the knowledge base.

Triage every prompt to one of three: answerable from the materials (a **check**), answerable only
with data they don't carry (a **question in Questions to validate**), or advice / needs-analysis
(**out of scope**).

## Compute

Only what a check requires; the server computes most figures — take them as-is.

- **Effective rates from `effectiveTaxRates.rates`, not a ratio you build.** Where
  `basisIdentified` is `false`, quote the dollars, not the percentage.
- **Establish whose income it is before any per-person ceiling** — Social Security wage base, the
  SS component of SE tax, the excess-withholding credit, contribution limits. Additional Medicare
  Tax is the exception that aggregates.
- **Marginal rate is computed** from `bracketDistribution` and the rate summary, carryforwards
  included — never from income size. Check state `structure` before calling anything marginal.
- **Each MAGI separately** — IRMAA MAGI (AGI + tax-exempt interest) ≠ NIIT MAGI. Name the
  provision beside every MAGI.
- **Ordinary taxable income** = taxable income less preferential income; bracket distances measure
  on that remainder.
- **Distance to a threshold** is a positive number with its direction in words, never signed.
- **A preferential-rate position carries its consequence.** Above the 0% capital-gains breakpoint,
  the household's qualified dividends and long-term gains bear 15% (or 20% above the upper
  breakpoint) — state that rate and the dollars it falls on, not merely that no 0% room remains.
- **The safe-harbor requirement is a reproduced prong, never a figure taken on faith** — the lesser
  of 90% of this year's tax or 100%/110% of last year's, built from the return and the constants. A
  "safe harbor" amount echoed on the lookback, or the sum of next year's 1040-ES vouchers, is a
  planning estimate, not the requirement; do not state a shortfall or cushion from it. If neither
  prong can be reproduced from the materials, give the return-confirmed penalty in dollars and route
  the withholding question to Questions to validate.

Distance and rate are position. Projected savings and the dollar effect of a change the household
did not make do not appear — that is the scenario-modeling handoff.

## Suppress

After detection, before writing. Every item passes four gates; drop whatever fails.

1. **Read gate** — was the figure actually read, and does it say what the item claims? An absence
   points at a line read and empty.
2. **Eligibility gate** — do all preconditions hold for *this* household (ages, enrollment,
   coverage, entity type, filing status)?
3. **Currency gate** — is every rule current for this tax year, from the constants tools?
4. **Consistency gate** — does the item agree with the return and with every other item? When the
   item *is* an inconsistency resting on a lone checkbox or boolean flag that the numeric lines
   contradict, the flag is the unreliable party — drop it as a probable extraction artifact rather
   than reporting it as a finding.

The gates decide whether an item renders; the specific recurring false positives are the
suppression rules in [references/knowledge-base.md](references/knowledge-base.md) — read them
before writing. Suppress any planning strategy not tethered to a client fact, and keep
observations and modeled outcomes separate.

## Output

Title: `# Tax Insights — {Family} Family — Tax Year {YYYY}`. Then these `##` sections, in order,
beginning at the first that has content — no preamble, no household or income recap. Omit any empty
section rather than padding. These are quick hitters: each item earns its place by prompting a
conversation the advisor can act on, then stops — the advisor digs deeper by asking.

Keep every item scannable — lead with the figures, and hold the prose to what an advisor needs to
verify the fact and grasp the lever; trim anything past that. The balance is tight: enough detail
for verification and understanding, but not exposition or hand-holding. Foreground the numbers over
the sentences that carry them.

| # | Section | Fed by | Rendering |
|---|---|---|---|
| 1 | Return anomalies | 1 | Figures or classifications that may need correction. One `###` per item: the line, what it shows, what it should show, what follows. First, because they may be errors |
| 2 | Planning opportunities | Planning pass | One `###` per opportunity in the format below. Conditional only — each tied to a client fact |
| 3 | Threshold & timing signals | 2, 3, 4 | Lead with the shadow-tax position grouped by driver (each threshold its own MAGI), then a table `Threshold \| This household \| Distance` covering every threshold the household is close to or over (a comfortably-clear threshold is not measured); then a short `###` per trigger that carries a consequence |
| 4 | Questions to validate | 2, and any check needing records the materials don't carry | Each a question, then one line on what prompts it |
| 5 | Where more is needed | 2 | Optional, and usually omitted. Only for an insight that looks applicable from the data but can't be fully stood up without a specific input the materials lack — name the insight and the one missing input, nothing more. Not a general limitations notice, and not a recital of what was reviewed |

The output carries no **Sources** or citation line — it ends at the last section that has content.
Tax-law thresholds still come only from the constants tools and never from memory, but the delivered
insights do not cite a publisher; the advisor reads figures, not footnotes.

**Shadow taxes** is the advisor-facing name for the AGI/MAGI-driven cliffs (NIIT, IRMAA,
phase-outs). Lead Threshold & timing signals with it, grouped by cause not figure — each provision
modifies AGI its own way, so the lead never states a single household MAGI.

**Planning opportunity format** — four parts, kept tight:

- **Why this surfaced** — the client-specific facts that activated it, provenance stated naturally,
  plus a brief clause on how the lever works so the advisor can judge whether it applies. Where the
  opportunity turns on a fact the materials don't establish — pretax balances exist, the business is
  a passthrough — state the confirmation the feasibility turns on ("to determine feasibility, confirm
  the household holds pretax IRA/401(k) balances") rather than labeling the fact *unknown* or
  assuming it. A sentence or two; the mechanics are the advisor's to ask about, not to spell out.
- **Conversation to have** — the focused questions that determine fit, feasibility, or client
  intent.
- **Useful documents** — records that would confirm the facts or complete the analysis. Omit the
  line when none is needed.
- **Possible next analysis** — the scenario or comparison that could be modeled once inputs are
  confirmed. No invented outcome; omit when there is nothing to model.

**Presentation.** State position, never direction. Questions to validate are phrased
as questions. Anomalies name the line ("the 1099-R reports $48,000 gross with no taxable amount and
no rollover notation"). Absolute figures only; pair every rate with its dollars and income basis;
name the provision beside any MAGI; one final figure per statement. Attribute to the document ("the
federal return shows"), never to a tool, field, or screen. Never carry a figure from the example —
it belongs to a real household and to that household alone. 

## Common issues

Brief; the deep handling is in the knowledge base.

- **Lookback returns a different tax year than named** → say so in the first line and confirm
  before continuing.
- **Response condensed** (`note` says so) → shadow-tax positions still work; a band-edge distance
  you can't compute is simply not surfaced, or goes under Where more is needed if it blocks an
  applicable insight.
- **A filer's DOB is missing** → age-gated checks are *unavailable*, not clean; stay silent unless
  the gap blocks an applicable insight, then name it under Where more is needed.
- **A constants tool has no data for the year** → search the named publishers; if still
  unverifiable, drop the item — never estimate or use a remembered value — and surface it only where
  it blocks an applicable insight.
- **A rate carries `basisIdentified: false`** → quote the dollars, not the percentage.
- **`find_client` returns multiple** → ask which; two similar households produce a review correct
  about the wrong person.
