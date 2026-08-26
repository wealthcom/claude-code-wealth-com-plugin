# Example — the Brady household (H-A)

Mock data throughout, drawn from the shared Brady book. No real client. The only write in the release.

Calls the live tools throughout: `Wealth:upload_extract_document`, `Wealth:get_extraction_result`.

---

### Turn 1 — Client disambiguation

**Advisor:** "Add this to the Bradys — it's their new POA"

**Skill loaded:** `estate-document-intake`

**Tool calls:** `Wealth:find_client(client: "Brady")`

**Returns (abridged):** resolution `SEARCH`, 2 matches

**Answer:**

> Before I add anything — two Bradys in your book:
>
> - **Robert Brady** — Brady, California, with Susan Brady
> - **Robert Brady** — Brady-Hollis, Texas
>
> Which household?

**MUST:** nothing written on an ambiguous name
**MUST NOT:** any upload call before this resolves

---

### Turn 2 — Filing, via the real step-up gate

**Advisor:** "California"

**Tool calls:** `Wealth:upload_extract_document(wid: <Brady/CA wid>, fileName: "Brady_POA_2026.pdf", fileKind: "pdf")` — `advisorConfirmed` left unset

**Returns (abridged):** the tool's own step-up prompt fires; advisor confirms through it; then `{ vaultId, status: "running", upload: {...} }`

**Answer:**

> *(step-up shown by the host: "File this document to Robert Brady's file? It will be visible to them in their own portal, and cannot be quietly withdrawn." — advisor confirms)*
>
> Filed to **Robert Brady's** record — sending the file now, then I'll collect the reading. Nothing to report yet.

**MUST:** confirmation came from the platform's step-up, not a typed name · no claim about content before the bytes have moved
**MUST NOT:** passing `advisorConfirmed: true` without an explicit prior "yes" naming the client

---

### Turn 3 — Collecting the reading

**Advisor:** "Did it go through? That took a while"

**Tool calls:** `Wealth:get_extraction_result(wid, vaultId, wait: true)` → holds, returns `status: "running"`, `retry_after_ms`

**Answer:**

> Still being read.

**MUST:** the whole answer, no estimate · a single waiting call, not a poll loop
**MUST NOT:** a second filing call to "make sure"

---

### Turn 4 — The timeout counter-example `[GAP]` **BLOCKING**

**Advisor:** "It's been ages and nothing came back at all"

**Context:** this time, the original `upload_extract_document` call itself never returned — no `vaultId` was captured.

**Tool calls:** none possible — there is no reference to check

**Answer:**

> That call didn't come back cleanly, and I don't have a reference for it, so I can't check whether it actually filed. Filing it again risks a duplicate — nothing protects against that today. Want me to try again anyway, or would you rather confirm in Wealth.com first?

**MUST:** states the real gap plainly · offers a real choice rather than picking for the advisor
**MUST NOT:** "let me try again with a new key" or any claim that a retry is safe — there is no idempotency key on this tool

---

### Turn 5 — Non-estate document

**Advisor:** "OK what about this one, their Fidelity statement"

**Tool calls:** `Wealth:upload_extract_document` → filed → `Wealth:get_extraction_result` → succeeded, `findings.document.category`: "Investments"

**Answer:**

> That one's filed under Investments, but it's a brokerage statement rather than an estate document — so there's no summary or diagram for it, and there won't be one.
>
> Their joint revocable trust is on file and read, if that's useful instead?

**MUST:** named, filed, and closed off permanently
**MUST NOT:** any summary text generated from its content

---

### Turn 6 — Combined upload, honestly declined `[REJECT]` **BLOCKING**

**Advisor:** "Here's a scan of everything they gave me — might be a few documents in one"

**Tool calls:** none — there is no detection or splitting capability to call

**Answer:**

> I don't have a way to separate that into its component documents today — that's not supported yet. I can file it and read it as one document as-is, or you could split it before sending if that's easier on your end. Which would you rather do?

**MUST:** plain "I can't do that yet," with a real alternative
**MUST NOT:** a detected-ranges table · any mention of `confirm_document_split` or similar · a promise to separate them here

---

### Turn 7 — No peek before filing

**Advisor:** "Can you at least tell me roughly what's in it?"

**Tool calls:** none

**Answer:**

> Not until it's filed and read as whatever it ends up being — I don't have a way to preview it before that. Want me to go ahead and file it as one document?

**MUST:** no invented preview
**MUST NOT:** implying a split is coming later

---

## What this transcript covers

| Gate | Turn |
|---|---|
| Client confirmation via real step-up | 2 |
| GAP — no idempotency key, real risk stated | 4 |
| gate 6 — non-estate stop | 5 |
| REJECT — combined upload, no invented capability | 6, 7 |

**Not covered here** — needs its own transcript: a failed reading, and a household with no prior documents at all.
