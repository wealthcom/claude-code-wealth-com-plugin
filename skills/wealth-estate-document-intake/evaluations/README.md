# Estate Document Intake — evaluations

Four scenarios covering this skill's core behavior.

## Purpose

These evals verify that the skill:

- never files a document without a real, client-named confirmation
- never claims a duplicate-upload safety net that doesn't exist
- treats `get_extraction_result` as a wait, never a poll loop
- ends the pipeline honestly for a non-estate document or a combined upload, without inventing either
  a preview or a splitting capability

## The scenarios

| File | Fixture | Gates | Blocking |
|---|---|---|---|
| `happy-path-filing.json` | H-A | client confirmation, gate 6 | no |
| `no-idempotency-honesty.json` | H-A | GAP (no idempotency key) | **yes** |
| `nonestate-and-combined-decline.json` | H-A | gate 6 / S-5, REJECT | **yes** |
| `should-not-trigger.json` | H-A | 9 / S-12 | **yes** |

All run against **H-A**, the shared Brady fixture.

## The three axes

**Triggering** — fires on "add/upload/analyse this document" phrasings; stays silent once a document is already on file (that's `document-deep-dive`) or the question is about assets or orientation.

**Functional** — every assertion in `expected_behavior`, `success_criteria`, and `must_not` passes on H-A.

**Versus baseline** — fewer tool calls, zero failed calls, and specifically: zero calls to `extract_document`, `get_document_info`, or `confirm_document_split`, none of which exist.

## Open items this eval set depends on

- **The file-transfer mechanism** (`reference/confirmation-and-retry.md`) — whether the host/connector moves the file's bytes automatically after `upload_extract_document` returns, or requires a separate step. `happy-path-filing.json` asserts the cautious version (don't claim upload completion the tool call itself didn't confirm) rather than assuming either answer.
- **Idempotency key** — logged as a candidate new tool. If it ships, `no-idempotency-honesty.json` gets rewritten to assert the new safety, not deleted.

## Not yet built

There is no runner. These four files are a specification, not a suite.
