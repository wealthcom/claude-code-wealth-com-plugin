# Document states

Check the state before promising anything. Each state has one correct answer.

**The real states.** `list_documents`' `readingState` on each row is exactly this four-value enum —
`awaiting_upload`, `reading`, `ready`, `unavailable`. There is no fifth value for "non-estate" and none
for "combined." Those two cases are real and you still need to handle them, but you infer them from
other signals, not from a status field — see below.

| State (real, on the row) | What it means | What to say |
|---|---|---|
| **awaiting_upload** | The file hasn't arrived | "Still waiting on that file" |
| **reading** | Still being read | "Still being read" — and nothing more |
| **ready** | Can be summarised | Proceed normally |
| **unavailable** | Could not be read | Say it failed and needs attention in the app |

| Inferred case (not a status field) | Signal | What to say |
|---|---|---|
| **Non-estate document** | The row's `documentType` label itself (e.g. "Investments") | Stop permanently. No artifacts, ever |
| **Combined upload** | Advisor raises it, or a document reads strangely | No splitting capability exists today — say so |
| **Unsupported type** | `get_flowchart` reports no deterministic diagram | Summary and terms still work. No diagram |

---

## Non-estate

The hardest one, because it's where being helpful becomes fabrication.

The document lives in the vault. It has no executive summary, no diagram, no observations — and never will. There is nothing to generate from its text. There's no dedicated flag for this on the row — `list_documents`' `documentType` column is the client file's own label (for example "Investments", "Wills & Supporting Documents"), and a plainly non-estate label is the signal to stop before calling anything else. If it's ambiguous, `get_executive_summary` and `get_estate_risks` will simply come back saying there's nothing on file for that document — which reads the same as "not analysed yet," so lean on the Type label first.

> "That one's filed under the client's investments — it isn't an estate planning document, so there's no summary or diagram for it. Want me to look at their trust documents instead?"

**Never:** generate a substitute from the document text · offer a lookback report for it · describe what a summary "would" contain · present the unsupported status as "still processing."

**Why it matters:** this is the single most likely place to produce fabricated estate analysis, and a fabricated summary is indistinguishable from a real one to the advisor reading it. `[gate 6 / S-5]`

---

## Reading

"Still being read" is the complete answer. No partial artifacts, no partial summary, no estimate of when.

The failure mode being prevented: a half-read document produces artifacts that look finished. Presenting them implies the analysis is done.

---

## Unavailable

Say it could not be read. Say it needs attention in the app. Do not present an unavailable document as still being read — that sends the advisor away to wait for something that will never arrive.

---

## Combined uploads — not supported today

There is no `confirm_document_split` tool. **`list_documents`' real schema carries no fields at all for detected document ranges or a combined flag** — checked directly against the row schema.

If an advisor raises this case (or a document's own content suggests it), the honest answer is that this isn't something this can do yet:

> "I don't have a way to separate that into its component documents today — that would need to happen in Wealth.com, if it's supported there. Want me to work with it as filed, or move to a different document?"

**Never** invent a splitting step, ranges, or a confirmation flow that isn't real. This is a fail-closed answer, not a wording problem to work around.

---

## Mirror documents

Spouses often have the same document executed twice — two healthcare directives, near-identical. **There is no detection surface for this at all.**

So: never assert two documents are mirrors, and never deduplicate them. Present both. If they read nearly identically, note the resemblance as an observation for the advisor to confirm, not as a finding.

> "Robert's and Susan's healthcare directives read almost identically — worth a look to confirm they're a matched pair."

`[S-8]`

---

## Unsupported type — the diagram case

An estate document with no deterministic diagram for its type. The summary and the terms still work; only the diagram is unavailable.

Say which document it is, say it doesn't have a diagram, and offer to continue on its terms.

> "The family ILIT doesn't have a disposition diagram — that view only covers certain trust structures. I can walk you through its terms instead: who the trustee is, who the beneficiaries are, and how distributions work."

**Never** substitute: no model-drawn diagram, no node-and-edge table, no prose description of the diagram that would have existed.

**The tension to be honest about:** the app shows advisors a diagram on more documents than this path covers. So this message will sometimes fire where the advisor has previously seen a picture in the app, for a document type without a deterministic diagram. Do not paper over it with a substitute. `[gate 3 / S-3]`

This tool is `get_flowchart(wid, vaultId)`, scoped per-document by `vaultId` — the tool description
itself says "a flowchart belongs to one document, so name the document rather than expecting one to
be picked," and "when no structure is on file the answer says so rather than showing an empty one."
So a "no diagram for this document" answer genuinely means this specific document's structure isn't
on file — not that there's no client-level structure at all. Most documents should have a diagram
available; the remaining minority correctly fail closed per the behavior above.
