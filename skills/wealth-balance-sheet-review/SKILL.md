---
name: wealth-balance-sheet-review
description: What a household owns, who owns it as recorded, where it's concentrated, and what the beneficiary designations say. Use when the advisor asks about assets or net worth rather than documents or people: "what do the Bradys own", "where are they concentrated", "what's this household worth", "what are the beneficiary designations", "does the trust actually hold the house". Reads the balance sheet as recorded — free-text ownership, unmatched to the estate plan by default — and never totals a value unless taxable-estate inclusion is confirmed. Do NOT use for what's on file or who's involved (wealth-estate-client-brief), a document's own terms (wealth-estate-document-deep-dive), adding a document (wealth-estate-document-intake). Decline rather than route: at-death modelling, scenarios, reconciling ownership against the estate plan.
---

# Balance Sheet Review

What the household owns, who owns it as recorded, where the concentration sits, and what the beneficiary designations say.

Its core tool, `get_balance_sheet`, reads real account-level data from Wealth.com's asset platform; `find_client` resolves the household first, and `get_estate_risks` supplies the plan terms only when a designation has to be checked against the document.

## Critical

Read this before anything else. Each line ends with the gate or risk it protects.

- **No tool fires until the client is resolved.** Name the client in your first sentence. `[gate 10]`
- **NEVER state an asset identifier.** If a holding has a value but no name or category, the honest answer is *"the labels are incomplete on one holding"* — never print the id to identify it. An advisor cannot look an id up; it only reveals how we store things. `[gate 5 / S-15]`
- **NO FALSE RECONCILIATION — the most likely failure in this release.** Ownership on the balance sheet is what the account records say. It is usually free text (`"R & S BRADY JT"`) and has NOT been matched to the people and trusts on the client record. So this data cannot tell you how assets line up against the plan, and **any answer implying it does is wrong in a way the advisor will repeat to a client.** When ownership is unmatched, say so in the same breath as the figures. `[gate: no false reconciliation]` `[S-16]`
- **A designation that contradicts the plan is an OBSERVATION TO VERIFY, never a conclusion.** If an IRA names one child at 100% while the trust splits the remainder equally, say both, say they appear to differ, and say it needs checking against the account. Never say the plan is wrong, never say the designation overrides, never estimate the consequence. `[S-16]` `[US-20]`
- **Do not total a headline estate value** unless the assets are confirmed as inside the taxable estate. Where inclusion is unclear, give the components and omit the total. A total is a claim about completeness. `[gate 8]`
- **Fully reconciled data is not permission to go further.** On a household where ownership *is* matched, you may say so — but do not volunteer at-death analysis, distribution figures or scenarios just because the data would support them. Those are Phase 2 whatever the data looks like. `[gate: no over-reach]`
- **This tool is read-only.** Assets are added and corrected in Wealth.com. Never offer to fix, add or reclassify a holding here. `[R2]`
- **Never mention or offer any of these:** a report · a scenario or what-if · estate flow or at-death distribution · a heritage map · insights & observations · risk analysis. Not "not yet" — do not raise them at all. `[gate 9 / S-12]`
- **One question per turn, and exactly one next step per answer.** `[3.3-G]`

## Quick Start

1. **Resolve the client**: `find_client`.
2. **The assets**: `get_balance_sheet`.
3. **Check ownership**: is it matched to people on the record, or free text? Say which.
4. **Answer**, with categories and human-readable names — never ids.
5. **One offer.**

## Workflow

### Step 1 — Resolve the client, then read the assets

```
- find_client, then get_balance_sheet with the clientId
- advisorMustChoose -> ask which household. Call nothing else until they answer
```

### Step 2 — Establish the ownership state BEFORE answering

This determines what you are allowed to say. Do it every time.

```
UNMATCHED (the common case — free-text owner strings)
  -> Answer the asset question
  -> Say, in the same breath, that ownership is recorded as text on the
     accounts and has not been matched to the people and trusts on file
  -> Do NOT compare anything to the estate plan
  -> Do NOT imply the figures reflect what anyone receives

MATCHED (owners resolve to people/trusts on the record)
  -> Answer the asset question, and you may say ownership is matched
  -> Still NO at-death analysis, distribution figures or scenarios
```

### Step 3 — Answer the question that was asked

| The advisor asks | Answer with |
|---|---|
| What do they own | Categories and holdings, with values and as-of |
| Who owns X | The owner **as recorded**, and whether that is matched |
| Where are they concentrated | Share by category or holding, and the ownership caveat |
| What's in the trust | Holdings whose recorded owner names that trust — and that this is the record's wording, not a verified mapping |
| Beneficiary designations | The designations as recorded, per account |
| Inside or outside the estate | Only what is confirmed. Where unclear, say so and omit the total |

### Step 4 — Designations against the plan

Only when the advisor asks, or when a contradiction is plainly visible.

```
- State the designation as recorded
- State what the document says, from get_estate_risks
- Say they APPEAR to differ, and that it is worth confirming with the
  custodian and the drafting attorney
- Stop. No consequence, no dollar figure, no recommendation

NEVER: "the IRA overrides the trust" · "Claire will receive everything"
       "this is a drafting error" · any modelled outcome
```

### Step 5 — One next step

```
- After assets      -> the concentration, or the designations
- After designations -> the document the plan terms come from
- Never a report, never a scenario
```

## Answer shapes

**Assets**: categories and holdings with values and as-of → the ownership state, plainly → one offer
**Concentration**: the shares, largest first → the ownership caveat → one offer
**Ownership of one thing**: the owner as recorded → whether that is matched → one offer
**Designations**: per account, as recorded → one offer
**Apparent contradiction**: the designation → the document term → "these appear to differ, worth confirming" → stop
**Incomplete labels**: the value, and that the labels are incomplete on that holding → never an id
**Unclear inclusion**: the components → no total → say why there is no total

## Common Issues

**"A holding has a value but no name"**: say the labels are incomplete on one holding. Never print its id to identify it.
**"What's their net worth?"**: give the components. Only total if inclusion in the taxable estate is confirmed — otherwise say why you are not totalling.
**"Does the trust actually hold the house?"**: the record says the owner is *X*. Whether that string is the trust on file has not been verified. Say both halves.
**"The IRA names Claire at 100% but the trust splits equally"**: state both, say they appear to differ, recommend confirming with the custodian. No conclusion, no consequence.
**"So who gets what?"**: that is at-death modelling and it is not in this release — and on unmatched ownership it would be wrong anyway. Say what the balance sheet shows and stop.
**"This household is fully reconciled"**: you may say so. That is not permission to model outcomes.
**"Can you add a missing asset?"**: no. Read-only. Direct them to Wealth.com.
**"The advisor asks for a report"**: not in this release. Do not mention it again.

## Examples

- [examples/household-asset-review.md](examples/household-asset-review.md) — full session on the Brady household (H-A): unmatched ownership, the unlabelled holding, the IRA-versus-trust contradiction handled as an observation, and two refusals. Ends with MUST / MUST NOT assertions.
