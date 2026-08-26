# Estate Document Deep Dive — evaluations

Four scenarios. Three of them gate the launch.

## Purpose

These evals verify that the skill:

- loads on single-document requests and stays silent on the other three skills' territory
- answers only from a document's own extracted artifacts, and never reasons a provision into existence
- renders a diagram when one exists and **fails closed** when one doesn't
- labels every document-derived answer as extracted and unconfirmed, with attribution and an as-of date
- never mentions a capability this release cannot deliver
- behaves consistently across Haiku, Sonnet, and Opus

## The scenarios

| File | Fixture | Gates | Blocking |
|---|---|---|---|
| `happy-path-jrt.json` | H-A | 10, 5 | no |
| `failclosed-ilit-diagram.json` | H-A | 3 / S-3 | **yes** |
| `refusals-provisions-flow-nonestate.json` | H-A | 6, 7, 9 / S-5, S-12, S-13 | **yes** |
| `should-not-trigger.json` | H-A | 9 / S-12 | **yes** |

All four run against **H-A** — assets present, nothing reconciled, trust cards unverified. That's the 88% case and the state where a complete-looking answer is most tempting and most wrong.

**Missing coverage, and it's known:** combined-PDF detection and mirror-pair handling need a scenario on **H-C**; the clean decline on a household with no documents needs one on **H-D**. Neither is launch-blocking for this skill, but both are real behaviour that currently has no test.

## The three axes

**Triggering** — ≈90% on the description's phrasings *and* paraphrases of them; 0% on the twenty queries in `should-not-trigger.json`.

**Functional** — every assertion in `expected_behavior`, `success_criteria`, and `must_not` passes on the named fixture.

**Versus baseline** — same tasks with the skill disabled: the skill must produce fewer tool calls, fewer turns, and **zero** failed calls. If it doesn't beat the baseline, the skill is overhead.

## How to read the schema

- **`expected_behavior`** — the *path*. Ordered, names tools and arguments, cites reference files. Encodes acceptable branches ("…or asks which document").
- **`success_criteria`** — the *artifact*. Counts, formats, attribution, provenance, and the tool sequence with call multiplicities.
- **`must_not`** — the prohibitions, written to be regex-checkable. **This is where the launch gates live.** Note that most of them grep the *answer text*, not the tool trace: a run can have a perfect call trace and still fail by mentioning estate flow in its closing offer.
- **`gates`** — ties back to §3.4 and §7 so coverage is auditable.
- **`fixture`** — one of the five households in §3.6. Never invent client data. If a scenario needs something the fixture lacks, extend the fixture.

## Writing criteria that work

**Good** — specific, testable:

- "Attributes the figure to 'the Brady joint revocable trust, executed June 4th 2019'"
- "Turn 2 contains zero dollar figures, zero percentages, and zero per-recipient amounts"
- "The final offer names the next document explicitly rather than saying 'another document'"
- "Tool sequence is find_client (1x) then list_documents (1x) then get_document_info (1x)"

**Bad** — vague, untestable:

- "Gives a good summary"
- "Handles the unsupported case appropriately"
- "Uses correct formatting"
- "Doesn't overstep scope"

## Open items this eval set depends on

- **D-5** — whether any verified-vs-unverified signal is readable. Until it is, "extracted and unconfirmed" is asserted as the universal default.

## Not yet built

There is no runner. These four files are a specification, not a suite — nothing executes them today, and the `must_not` regexes have no harness. **Someone needs to own that**, and it isn't in §6 of the PRD yet.
