# Document outcomes

## `Wealth:get_extraction_result` — a wait, not a poll

One call holds until the reading settles (`succeeded`/`failed`) or its budget runs out (`running`,
with a `retry_after_ms`). The tool's own description forbids calling it "back to back or in a tight
loop" — leave `wait` unset so it holds, and only call again, after the interval given, if it's still
`running`. There is no separate status-lookup tool to poll.

| Outcome | Say |
|---|---|
| `running` | "Still being read." Nothing else — no partial content, no estimate |
| `succeeded` | Offer the summary, by document name |
| `failed` | It has ended; needs attention in Wealth.com. Never "still being read" |

> "Still being read."

> "That's filed and read — want the summary for it?"

> "That one didn't make it through the reading — it'll need attention in Wealth.com before there's
> anything to summarise."

## Non-estate documents

There's no dedicated flag for this on the reading result. `findings.document.category` (or
`list_documents`' `documentType`) carries the client file's own label — a plainly non-estate label
(e.g. "Investments") is the signal to stop. Say where it's filed, that no summary will ever be
produced for it, and offer an estate document instead.

## Combined uploads — not supported today

There is no `confirm_document_split` tool, and no field anywhere in the current schemas for
detected document ranges. If an advisor's upload looks like several documents bound together, say
plainly that this can't separate it today. Offer to file and read it as one document as-is, or
suggest the advisor split it before sending. Never invent a detected-ranges table or a confirmation
flow for a capability that doesn't exist.

## A newly filed document that may duplicate one already on file

`upload_extract_document` has no dedup check either — filing doesn't compare the new document against
what's already in the client's file. If the advisor is filing something that sounds like it might
already be there (a restated trust, a second copy of a directive), that's the same "mirror documents"
situation `wealth-estate-document-deep-dive` handles on the read side (see its
`reference/document-states.md`): file it anyway, never decline to file on a hunch that it's a
duplicate, and never assert the two are the same document yourself. Once it's read, say plainly that it
resembles one already on file and let the advisor confirm — don't silently merge, and don't skip
filing it.

> "That one's filed and read. Worth flagging — it reads a lot like the healthcare directive already on
> file for them. Want to take a look at both side by side?"
