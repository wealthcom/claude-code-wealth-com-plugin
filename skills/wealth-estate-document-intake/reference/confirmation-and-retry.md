# Confirmation and retry — procedure and why

## The confirmation gate

`Wealth:upload_extract_document` takes `advisorConfirmed: boolean`. Two paths:

1. **Leave it unset (preferred).** The tool itself calls a step-up confirmation before filing anything
   — this is part of the platform's write-consent design for filing anything new (OAuth step-up scope
   + consent for writes), not a UI toy. Prefer this path whenever the host can render it.
2. **Pass `advisorConfirmed: true`** only when the advisor has explicitly said to go ahead **in this
   conversation, naming the client** — not a bare "yes," not an earlier mention of the client's name,
   not impatience. This bypasses the platform's own step-up, so treat it as the exception, not the
   default.

**Why the gate is procedural, not a name check.** The real gate on `upload_extract_document` is a
step-up prompt or an explicit boolean — never a string comparison against a typed client name. Either
way, the underlying goal is the same: nothing files without a real confirmation naming the client.

## When confirmation is declined or unclear

The client is already resolved by this point — Step 1 handled that — so a declined or unclear
confirmation is never a "which household" question. It's a "proceed or not" question. If the advisor
says no, or hedges, or the confirmation just isn't there yet: nothing is written, and the honest next
question is whether they want to go ahead now or come back to it, not a re-ask of who the document is
for.

> "Not filing anything until you confirm — want me to go ahead with the Bradys' POA, or hold off?"

Silence is not a yes. An earlier mention of the client's name, from a different part of the
conversation, is not a yes either — the confirmation has to be about filing this document, given now.

## Why there is no duplicate-upload protection

`upload_extract_document` has no `idempotencyKey` field. If a call returns a `vaultId`, you have a
reference to check with `get_extraction_result` before assuming anything failed. If a call times out
or errors **before** any `vaultId` comes back, there is nothing to check — there is no way to tell
you whether that upload landed. This is a real product gap, not something this skill can work
around. Say the real risk plainly
rather than inventing language ("the same key," "it's protected") that implies a safety net that
doesn't exist.

## The file-transfer shape — open question

`upload_extract_document` returns a `vaultId` and a presigned upload URL with required headers; it
does not take the file's bytes as an argument. Something has to PUT those bytes to the URL before
there's anything to read — by design, out of band, never through this tool or the model's context. Open
question: whether that happens automatically via the host/connector layer, or as a distinct step the
advisor or the app performs — a runtime/product question, not a source-reading one. Don't tell the
advisor a document is "uploaded and being read" until you have real confirmation the bytes moved — the
tool call returning is not that confirmation by itself.
