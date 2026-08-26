# Presentation formats

Format is part of an answer's correctness — reused from the pattern the other estate skills share.

## Exact columns

**Client candidates during disambiguation**
`Client · Family name · State · Spouse`
Nothing else — no email, phone, address, DOB, or any identifier. `[gate 5]`

**Document inventory** (`Wealth:list_documents`)
`Document · Type · Upload date · Document date`
No Status column. Dates as `YYYY-MM-DD`, no times. A document whose `readingState` isn't `ready` is
named in prose beneath the table, never as a fifth column.

## Which shape

| Content | Shape |
|---|---|
| Client profile facts | Prose — a small, fixed set of fields, tabulating them is overkill |
| Client candidates | Table (see above) |
| Document inventory | Table (see above) |
| "The record indicates children" | One clause in prose — never a table, never a name |

## When candidates look identical on every permitted field

Two candidates can share family name, state, and spouse and still be different clients — the fourth
permitted field doesn't always break the tie. Never fall back to showing a `wid` or any other
identifier to distinguish them, even here. Say plainly that they look identical on everything you can
show, and ask a question that only the advisor can answer:

> "Two Robert Bradys come up, both in California with a spouse named Susan — I can't tell them apart
> from what I'm able to show you. Is there anything else that would tell them apart, like a middle
> name or a different city?"

If the advisor has no further distinguishing detail, say so and stop rather than guessing — this is a
real ceiling on the disambiguation fields, not a formatting problem.

## Missing or unknown profile fields

`find_client`'s DIRECT payload doesn't guarantee every field is populated — state, marital status, or
the children/pets indicators can come back null on a thin record. Say what's known, and say plainly
that the rest isn't on file rather than omitting it silently or guessing a plausible-sounding default:

> "Robert Brady, California. Marital status and whether there are children aren't on file for this
> client."

Never infer marital status from the presence of a spouse's name elsewhere, and never infer children
from a trust naming remainder beneficiaries — both would be reasoning past what the client record
itself states.

## Attribution and as-of

Every profile fact and every document row carries what it came from, in a form an advisor recognises
— never a `wid`, `vaultId`, or other identifier. `find_client`'s DIRECT payload doesn't carry its own
as-of date the way a document read does; state facts as "on file" rather than inventing a date that
isn't there.

## Open — worth revisiting

The client record's boolean indicators (has-children, marital status) and the total absence of named
beneficiaries are the ceiling on what this skill can say about "who's involved" today — not a design
choice this skill's format rules can work around. If a future release ships fuller spouse/children/
beneficiary detail on the client record itself, this file's "which shape" table gets a new row; until
then, treat the absence as final for every answer, not just the first one you hit.
