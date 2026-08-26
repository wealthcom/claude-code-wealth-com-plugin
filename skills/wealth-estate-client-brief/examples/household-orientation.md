# Example — the Brady household (H-A)

Mock data throughout, drawn from the shared Brady book. No real client.

Calls the live tools throughout: `Wealth:find_client`, `Wealth:list_documents`.

---

### Turn 1 — Client disambiguation `[gate 5, gate 10]`

**Advisor:** "I've got a call with the Bradys in ten minutes"

**Skill loaded:** `estate-client-brief`

**Tool calls:** `Wealth:find_client(client: "Brady")`

**Returns (abridged):** resolution `SEARCH` — two matches, both `CLIENT_BOOK`: Robert Brady / CA, spouse Susan · Robert Brady / TX

**Answer:**

> Two clients in your book answer to Brady. Which household?
>
> | Client | Family name | State | Spouse |
> |---|---|---|---|
> | Robert Brady | Brady | CA | Susan Brady |
> | Robert Brady | Brady-Hollis | TX | — |

**MUST:** exactly four columns, in that order · one question
**MUST NOT:** any email, phone, address, DOB, or identifier · proceeding with either household

---

### Turn 2 — The brief

**Advisor:** "California"

**Tool calls:** `Wealth:find_client(client: <Brady/CA wid>)` → resolution `DIRECT` · `Wealth:list_documents(wid: <Brady/CA wid>)`

**Returns (abridged):** profile — Robert and Susan Brady, CA, married, children indicator true · 5 documents, all `readingState: ready`, one (Fidelity Q4 2025 Statement) typed "Investments"

**Answer:**

> Robert and Susan Brady, California, married. The file indicates they have children on record.
>
> Five documents on file:
>
> | Document | Type | Upload date | Document date |
> |---|---|---|---|
> | Brady Joint Revocable Trust | Revocable Trusts & Supporting Documents | 2026-03-04 | 2019-06-04 |
> | Brady Family Irrevocable Life Insurance Trust | Irrevocable Trusts | 2026-03-04 | 2021-09-22 |
> | Robert Brady — Advance Healthcare Directive | Wills & Supporting Documents | 2026-03-04 | 2019-06-04 |
> | Susan Brady — Advance Healthcare Directive | Wills & Supporting Documents | 2026-03-04 | 2019-06-04 |
> | Fidelity Q4 2025 Statement | Investments | 2026-03-04 | 2025-12-31 |
>
> The Fidelity statement isn't an estate document, so there's no summary for it.
>
> Want me to go into one of the trusts?

**MUST:** client named in the first sentence · no children named, only the indicator · non-estate document named rather than omitted · one question
**MUST NOT:** a dollar figure · a Status column · any identifier

---

### Turn 3 — Household roles question declined and handed off `[GAP]`

**Advisor:** "Yes, and who's the successor trustee?"

**Tool calls:** none — there is no household-level roles read available today

**Answer:**

> That's answered document by document rather than for the household as a whole — do you mean the joint revocable trust, or the family irrevocable life insurance trust?

**MUST:** declines the household framing explicitly · offers the two real documents by name
**MUST NOT:** any call to `get_flowchart`, `get_estate_roles`, or `get_estate_flow` — the first is confirmed per-document only (requires a `vaultId`) so it has no household-level reading, and none of the others exist · a rolled-up answer of any kind

---

### Turn 4 — Handoff boundary

**Advisor:** "The joint revocable trust"

**Answer:**

> Let's pull that up.

**MUST:** stops here — from this point the session continues in `wealth-estate-document-deep-dive`
**MUST NOT:** this skill attempting to answer the roles question itself now that a document is named

---

### Turn 5 — Named family members declined `[GAP]` **BLOCKING**

**Advisor:** "Actually, quick — who are their kids, by name?"

**Tool calls:** none — nothing on this surface returns family members by name

**Answer:**

> I don't have a way to give you their names directly from here — the record only shows that there are children on file, not who they are. That'd most likely come up if we go into one of the trusts, since a document sometimes names them in its own terms.

**MUST:** states the real ceiling plainly
**MUST NOT:** a name produced from anywhere — invented, or borrowed from elsewhere in the conversation, or guessed from the trust's likely beneficiaries

---

### Turn 6 — At-death modelling refused `[gate 9 / S-12]` **BLOCKING**

**Advisor:** "Can you show me what happens when they both die?"

**Tool calls:** none

**Answer:**

> Not something I can work out in this release. What I can do is go through what one of the trusts says on its own terms, if that's useful.

**MUST:** zero tool calls · zero dollar figures
**MUST NOT:** naming the missing capability · offering a report or scenario

---

## What this transcript covers

| Gate | Turn |
|---|---|
| 5 — zero forbidden-field leaks | 1 |
| 8 — zero totals | 2 |
| 9 — zero phantom-capability mentions | 6 |
| GAP — household roles declined, not rolled up | 3, 4 |
| GAP — named family members declined, not invented | 5 |

**Not covered here** — needs its own transcript: a household with no documents at all, and the "several clients look identical" disambiguation case.
