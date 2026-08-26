---
name: wealth-estate-document-intake
description: Add an estate planning document to a client's file and have it read. Use when the advisor has a document in hand: "add this trust to their file", "can you analyse this", "here's the will they just sent", "upload this to the Bradys", "what's in this document", "I've got their POA". Confirms which client the document belongs to before anything is filed, collects the reading once, and handles the awkward cases honestly — a document that fails, one that turns out not to be an estate document. Do NOT use for documents already on file (wealth-estate-document-deep-dive), who is involved (wealth-estate-client-brief), assets (wealth-balance-sheet-review). Not for brokerage statements or tax returns. Decline rather than fake: separating a combined upload, guaranteeing no duplicate filing on a retry.
---

# Estate Document Intake

You get an advisor's document filed under the right client and read, and stay honest when it goes sideways.

**This is the only write in the release.** Filing a document publishes it into that client's own portal and cannot be quietly withdrawn.

The real gate mechanism and why there's no duplicate-upload protection are in
[reference/confirmation-and-retry.md](reference/confirmation-and-retry.md).

## Critical

Read this before anything else. Each line ends with the gate or risk it protects.

- **Client confirmation is not waivable, but the mechanism is a step-up, not a checked name.** Leave `advisorConfirmed` unset and let the tool's own step-up ask; only self-attest `true` on an explicit, client-named "yes" in this conversation. `[gate: client confirmation]`
- **There is no idempotency key.** A timeout with no `vaultId` in hand means there's no way to check whether it landed — say the real duplicate-filing risk plainly. `[GAP]`
- **Each call to `Wealth:upload_extract_document` files a new document.** To check on one already sent, call `Wealth:get_extraction_result` — never the upload tool again.
- **The file's bytes move out of band, not through this tool.** Don't say a document is "uploaded and being read" until you have real confirmation the bytes moved, not just that filing returned. See `reference/confirmation-and-retry.md`.
- **`Wealth:get_extraction_result` waits — it does not want to be polled.** One call holds; only call again, after its interval, if still `running`. `[correctness]`
- **A failed or unsupported document has ended.** Never "still being read," never partial results. `[gate 6]`
- **Estate documents only.** Not brokerage statements, not tax returns — say where it's filed and that no summary will ever exist for it. `[gate 6 / S-5]`
- **This cannot split a combined upload today.** Say so plainly; don't describe `confirm_document_split` — it doesn't exist. `[REJECT]`
- **Never offer to correct extracted data in conversation.** Hand off to Wealth.com. `[R2 → tool description]`
- **Never mention a report, scenario, estate flow, heritage map, insights, or risk analysis.** `[gate 9 / S-12 → tool description]`
- **One question per turn, exactly one next step.** `[3.3-G → tool description]`

## Quick Start

1. **Resolve the client**: `Wealth:find_client`. Never upload against a guess.
2. **Confirm** — prefer the tool's own step-up; self-attest only on an explicit, client-named yes.
3. **File it**: `Wealth:upload_extract_document { wid, fileName, fileKind }`.
4. **Collect the reading**: one call to `Wealth:get_extraction_result { wid, vaultId }` — let it wait.
5. **When it lands**, offer the summary — hand off to `wealth-estate-document-deep-dive`.

## Workflow

### Step 1 — Resolve the client, and mean it

```
- Wealth:find_client with what the advisor said
- unresolved -> ask which household. Upload NOTHING until they answer
- zero       -> say so. Never upload to the nearest name
```

A wrong read is recoverable. A wrong upload is visible to a client and cannot be quietly withdrawn.

### Step 2 — Confirm before filing

Prefer leaving `advisorConfirmed` unset. Only pass `true` on an explicit, client-named confirmation given in this conversation. If declined or unclear: nothing is written — say so, and ask whether they want to go ahead or hold off, rather than treating silence as a yes.

### Step 3 — File it

```
- Wealth:upload_extract_document { wid, fileName, fileKind, advisorConfirmed? }
- Returns immediately: vaultId, status "running", an upload step
- The bytes still have to move before there's anything to read — confirm
  that before telling the advisor it's filed and being read
- Timed out with a vaultId  -> check Wealth:get_extraction_result first
- Timed out with no vaultId -> no way to check; say so, and name the
  real duplicate-filing risk plainly
```

See [reference/confirmation-and-retry.md](reference/confirmation-and-retry.md).

### Step 4 — Collect the reading

```
- Wealth:get_extraction_result { wid, vaultId } — leave wait unset
- running   -> "still being read," nothing else; wait the given
               interval before asking again, never immediately
- succeeded -> offer the summary, by document name
- failed    -> it has ended; needs attention in Wealth.com
```

See [reference/document-outcomes.md](reference/document-outcomes.md).

### Step 5 — The awkward cases

Non-estate: say where it's filed, that no summary will ever exist for it, offer an estate document instead. Combined upload: say this can't separate it today; offer to file as-is or have the advisor split it first. See `reference/document-outcomes.md` for both.

### Step 6 — Hand off

Once read, offer the summary for that document, by name. `wealth-estate-document-deep-dive` owns the conversation from there. One question — never the whole plan or a report.

## Answer shapes

**Before filing**: which client, named → the confirmation → nothing written until it comes back
**Filed**: being read → one offer to check back
**Still being read**: "still being read" → nothing else
**Succeeded**: ready → offer its summary by name
**Failed**: ended, needs Wealth.com attention → offer another document
**Not an estate document**: what it is, where filed, no summary ever → offer an estate document
**Combined upload**: can't be separated today → offer to proceed as-is or split first
**No `vaultId` after a timeout**: say so plainly, name the real duplicate risk

## Common Issues

**"They said yes"**: to what? If it didn't name the client, it isn't confirmation.
**"The upload seemed to time out"**: check `Wealth:get_extraction_result` if you have a `vaultId`; if not, say plainly there's no way to check and name the duplicate risk.
**"It's been being read a while"**: "still being read" is the whole answer — wait for the given interval before asking again.
**"It failed — try again?"**: it has ended; re-filing is a judgment call given no duplicate protection — say that explicitly rather than just doing it.
**"It's a brokerage statement"**: say where it's filed, no summary ever, don't describe its contents.
**"One PDF is three documents"**: can't be separated today — say so plainly.
**"Can you fix the extracted name?"**: no — Wealth.com only.

## Examples

- [examples/document-filing-session.md](examples/document-filing-session.md) — full session on the Brady household (H-A): the step-up confirmation, a timeout with no `vaultId`, a non-estate document, an honest decline on a combined upload, and two refusals.
