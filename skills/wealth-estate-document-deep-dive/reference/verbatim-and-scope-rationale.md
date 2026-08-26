# Why the verbatim and scope rules exist

Background for the Critical block's prohibitions — moved here so the block itself stays short. Read
this once; the imperatives in `SKILL.md` are what to actually follow on every call.

## Why reproduce, never rewrite

The article/page/block citations on a summary or an observation are how a regulated advisor verifies
a statement against the source document. A paraphrase drops them, or restates them loosely enough that
they no longer point at anything checkable. Once that's gone, the advisor has no way to tell a real
extracted claim from a plausible-sounding one — and a plausible-sounding one is exactly what an LLM is
good at producing. Reproducing the block exactly, citations intact, is the only version of this answer
that's defensible in front of a client.

## Why observations are never a legal review

`get_estate_risks`' own real tool description says these are AI-generated observations that "may vary
from one week to the next," not a settled legal judgment — and the absence of a flagged issue on one
document says nothing about whether the estate plan as a whole has gaps. Presenting them as findings,
or letting a clean read imply soundness, overstates what a fixture-derived Insights & Observations
enrichment actually is. "Insights" and "observations" are the tool's own internal names for this
capability — advisor-facing language should describe what's being offered ("anything flagged as worth
raising"), not name the underlying feature.

## Why a non-estate document stops the pipeline permanently

This is the single most likely place to produce fabricated estate analysis: nothing stops a model from
reading a brokerage statement's text and writing something that looks like an estate summary. A
fabricated summary is indistinguishable from a real one to the advisor reading it, so the only safe
rule is to never attempt one for a document the client file's own Type label says isn't an estate
document.

## Why this skill never totals a value

This skill reads one document at a time. A total implies a check against everything the client owns,
which is the balance sheet's job, not this one's — and ownership on the balance sheet is frequently
unmatched to the people and trusts on record (see `wealth-balance-sheet-review`), so a total assembled
here would look more authoritative than the underlying data supports.
