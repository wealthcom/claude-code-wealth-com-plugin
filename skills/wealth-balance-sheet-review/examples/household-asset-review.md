# Example — the Brady household (H-A)

Mock data throughout, drawn from the shared Brady book. No real client.

H-A is the primary fixture and the hard case: **assets present, nothing reconciled.** Ownership is free text, trust cards are unverified, and there is a deliberate designation contradiction. This is the 88% case, and the state where a reconciled-*looking* answer is most tempting and most wrong.

---

**Advisor:** what do the Bradys own?

`find_client` → resolved · `get_balance_sheet { clientId: "wid_8f21" }`

> **Robert and Susan Brady** — here's what's on the balance sheet, as of the latest account data.
>
> | Category | Holding | Owner as recorded | Value |
> |---|---|---|---|
> | Real estate | Primary residence, Palo Alto | R & S BRADY JT | $3,850,000 |
> | Investments | Brokerage — Fidelity | ROBERT BRADY | $2,140,000 |
> | Retirement | IRA | ROBERT BRADY | $1,620,000 |
> | Business | Northmoor Holdings LLC | ROBERT BRADY | $4,400,000 |
> | Investments | *(labels incomplete on this holding)* | — | $214,000 |
>
> One thing to flag: ownership here is the text on the accounts — "R & S BRADY JT", "ROBERT BRADY" — and it hasn't been matched to the people and trusts on their record. So this shows what's owned, but it can't tell you how any of it lines up against the trust.
>
> Want the concentration picture, or the beneficiary designations?

*Ownership state stated in the same breath as the figures, not as a footnote. The unlabelled holding is named as unlabelled — no id. No total.*

---

**Advisor:** what's the total?

> I can give you the components, which come to about $12.2m across those five holdings — but I'd stop short of calling that the estate value. Whether the business interest and the residence sit inside the taxable estate hasn't been confirmed here, and the trust hasn't been matched to any of these accounts. The number would look more settled than it is.

*Components yes, headline total no, with the reason. This is the difference between a figure and a claim.*

---

**Advisor:** where are they concentrated?

> Two holdings carry most of it — the business interest at Northmoor Holdings is the largest single position, with the residence next. Between them that's roughly two thirds of what's on file.
>
> Same caveat as before: this is the account records' view, not a reconciled one.

*The caveat repeats. It is not a one-time disclosure — every answer drawn from unmatched ownership carries it.*

---

**Advisor:** what about the beneficiary designations?

> The IRA names **Claire Brady at 100%**. The brokerage account has no designation recorded.
>
> Worth flagging: the 2019 joint revocable trust divides the residue **equally between Claire and Peter**. The IRA designation and the trust terms appear to point different ways. Designations sit with the custodian and pass outside the trust, so this is worth confirming with the custodian and with whoever drafted the plan — I can't tell from here which reflects current intent.
>
> Want me to pull up what the trust says on the residue?

*The whole gate in one answer. Both stated, described as appearing to differ, routed to the two people who can settle it. No conclusion about which wins, no consequence, no figure.*

---

## The counter-example — what must never happen

**WRONG:**

> The IRA designation overrides the trust, so Claire will receive the full $1.62m IRA plus her half of the residue — about $2.4m more than Peter.

Every clause is a fabrication: the override is asserted, the arithmetic implies matched ownership that does not exist, and the whole thing reads as a finding an advisor would take to a client. **This is S-16, and it is the most likely serious failure in this release** — because 88% of clients have numbers to show and none of the guardrails are visible in the output.

**RIGHT:** the answer above. Both facts, "appear to differ", and who to ask.

---

**Advisor:** so where does everything actually land at death?

> That's not something I can work out in this release — and on this household it would be misleading even if it were, because none of these accounts are matched to the trust. What I can do is stay on what's owned, or go into what the documents say. Which is more useful?

*Two reasons, both true, neither describing a capability that exists elsewhere.*

---

**Advisor:** can you add the vacation property? It's missing.

> That one has to go in on the Wealth.com side — I can read the balance sheet here but not change it. Once it's added it'll show up in this view.

*Read-only. R2: structuring belongs in the app.*

---

## Assertions

**MUST**

- `find_client` before `get_balance_sheet`
- The ownership state stated on EVERY answer drawn from asset data
- The unlabelled holding described as having incomplete labels
- Components given without a headline total, with the reason
- The IRA/trust difference stated as an apparent difference, with both sources
- A route to the custodian and the drafting attorney
- Exactly one question per turn

**MUST NOT**

- Any asset identifier in the answer text
- Any `clientId`, `userId`, `contactCardId` in the answer text
- A headline estate total on unconfirmed inclusion
- The words "overrides", "will receive", or any modelled at-death figure
- Any statement that ownership matches the trust
- An offer to add, correct or reclassify a holding
- The words report, scenario, what-if, estate flow, heritage map, risk analysis, insights
