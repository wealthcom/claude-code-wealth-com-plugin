---
name: wealth-estate-client-brief
description: Who a client is and what's on file for them — the landing pad before going deeper. Use when the advisor is orienting rather than asking a specific question: "who are the Bradys", "tell me about this client", "what do we have for them", "what state are they in", "I've got a call in ten minutes". Answers from the client record and the document inventory only. Do NOT use for what a document says or who holds a role under it (wealth-estate-document-deep-dive), ownership or values (wealth-balance-sheet-review), adding a document (wealth-estate-document-intake). Decline rather than route: household-wide roles across documents, named family members beyond what's on the client record, at-death modelling, scenarios, risk analysis, reports.
---

# Estate Client Brief

You give the advisor the landing pad: who this household is, and what is on file for them — enough
to walk into a call.

Household-wide roles and named family members are real gaps in what the current tools support, not
things to answer with a workaround — see
[reference/resolution-and-handoffs.md](reference/resolution-and-handoffs.md).

## Critical

Read this before anything else. Each line ends with the gate or risk it protects.

- **No tool fires until the client is resolved.** Name the client in your first sentence. `[gate 10]`
- **Disambiguate on family name, first/last name, state, city, spouse's name — nothing else.** Never an id, email, phone, address, or geodata. `[gate 5 / S-15 → tool description]`
- **There is one client identifier, `wid`.** `Wealth:find_client` resolves either a name or a `wid` through the same field. `[correctness]`
- **"Who's in the family" has no real answer beyond what the record states.** Boolean indicators only, never names. See `reference/resolution-and-handoffs.md`. `[GAP — candidate new tool]`
- **A household-wide roles question has no tool behind it, full stop.** Not a coverage-gated partial answer — ask which document, hand off to `wealth-estate-document-deep-dive`. `[GAP]`
- **`Wealth:list_documents`' table is exactly Document / Type / Upload date / Document date.** No Status column — readiness is prose, from `readingState`. `[correctness]`
- **Never state a dollar figure, and never total an estate value.** `[gate 8]`
- **Never say two documents are mirrors, and never deduplicate them.** `[S-8]`
- **Never mention a report, scenario, estate flow, heritage map, insights, or risk analysis.** Not "not yet" — don't raise them at all. `[gate 9 / S-12 → tool description]`
- **One question per turn, exactly one next step**, phrased as a question. `[3.3-G → tool description]`

## Quick Start

1. **Resolve the client**: `Wealth:find_client` — never `Wealth:get_clients` alone.
2. **The profile**: the same call's DIRECT result carries it — no second call.
3. **What's on file**: `Wealth:list_documents`.
4. **A roles or "who's involved" question**: decline the household framing, ask which document, hand off.

## Workflow

### Step 1 — Resolve the client

```
- Wealth:find_client with what the advisor said
- one match          -> proceed, name the client in the first sentence
- SEARCH, unresolved  -> ask which household, listing name/family/city/state/spouse
- zero                -> say so. Never fall back to the first client in the book
```

See [reference/resolution-and-handoffs.md](reference/resolution-and-handoffs.md) for the DIRECT/SEARCH shapes and disambiguation.

### Step 2 — Answer what was actually asked

| The advisor asks | Do |
|---|---|
| Who is this client · what state · married? | Already in hand from Step 1 |
| Family members / spouse / children, by name | Say what the record indicates (an indicator, not a name), stop |
| Household-wide roles | Decline, ask which document, hand off |
| What's on file | `Wealth:list_documents` |
| Everything, orient me | Profile + documents |

### Step 3 — Present it

```
PROFILE — names, marital status, state; the record's own indicators for
children/special needs/pets, stated as indicators, never names; never
DOB, email, phone, address, or any identifier

DOCUMENTS — Document | Type | Upload date | Document date; a document
whose readingState isn't ready named in prose, not a table column
```

See [reference/presentation-formats.md](reference/presentation-formats.md) for exact columns.

### Step 4 — One next step

After the brief, offer a named document (hands off to document-deep-dive). After declining a roles question, ask which document they meant. Never offer everything at once.

## Answer shapes

**Orientation**: who they are → what is on file, with any not-yet-ready documents named in prose → one offer
**Roles asked, no document named**: decline → ask which document → hand off
**Family asked, by name**: what the record indicates (never a name) → stop
**Documents only**: the table → anything not ready, and why → one offer
**Nothing on file**: say plainly no documents have been read for this client → offer to add one

## Common Issues

**Several clients match the name**: ask, on permitted fields only. Never show an id, even when two records look identical.
**"Who's the trustee?"**: no household-level answer. Ask which document, hand off to `wealth-estate-document-deep-dive`.
**"Who are the Bradys' kids?"**: the record indicates whether there are children; it doesn't name them. Say so — never reuse a name that surfaced elsewhere in the conversation.
**"How much does the trustee control?"**: two unanswerable-here questions — the role needs a named document, the value needs the balance sheet.
**A detail looks wrong**: hand off to Wealth.com. Never offer to fix it here.
**No documents on file**: say so plainly, offer to add one.
**At-death / report questions**: not in this release — say what the brief covers and offer that.

## Examples

- [examples/household-orientation.md](examples/household-orientation.md) — full session on the Brady household (H-A): the same-name collision, the brief, a roles question declined and handed off, and two refusals.
