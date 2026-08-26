# Wealth.com Claude Code Plugin

Six skills for financial advisors working estate planning, balance sheet, and tax cases in
Claude, backed by Wealth.com's live client data through the `wealthcom-mcp` connector.

**Estate**

- **wealth-estate-client-brief** — orient on a household: who the client is, and what's on file
  for them.
- **wealth-estate-document-deep-dive** — go deep on one estate document: its executive summary,
  disposition diagram, flagged observations, and follow-up questions about its own terms.
- **wealth-estate-document-intake** — file a new document to a client's record and have it read.

**Balance sheet**

- **wealth-balance-sheet-review** — what a household owns, who owns it as recorded, where it's
  concentrated, and what the beneficiary designations say.

**Tax**

- **wealth-tax-executive-summary** — a descriptive overview of a client's filed tax return:
  household and filing, income, deductions, tax and rates, capital-gains and IRMAA positioning,
  payments and safe harbor, and what's in the return.
- **wealth-tax-scenarios** — build and model a tax position for a year that hasn't been filed,
  either from the advisor's own assumptions (optionally sharpened by a document dropped into the
  conversation) or by rolling a filed return forward with changes.

Each skill ships its own `reference/` (domain rules and edge cases), `examples/` (an end-to-end
transcript), and `evaluations/` (triggering, functional, and should-not-trigger test cases).

## Requirements

These skills call tools exposed by Wealth's `wealthcom-mcp` server at
`https://mcp.wealth.com/mcp` (bundled in `.mcp.json`). You'll need a Wealth.com / C4FA advisor
account with that connector authorized — the skills themselves do nothing without it.

## Scope

This bundle covers reads, document intake, and tax modeling. It does not include the retired
plan-overview skill or at-death/estate distribution modeling. Skills bind to tools by description
rather than a pinned tool list, so behavior tracks whatever the connected server currently exposes
under those names.

**Two known gaps in the tax skills, both fail-closed rather than papered over:**

- **No scenario promotion yet.** `wealth-tax-scenarios` models are session-only today. The live
  registry has no tool to save or commit a modeled scenario to a client's profile. Until one
  exists, the skill says so plainly if asked to save one — it does not claim to persist anything.
  The corresponding eval (`save-scenario-declined.json`) will need rewriting to a real happy path
  once that capability ships.
- **No unrecaptured §1250 gain input.** Neither tax calculator has a field for depreciation
  recapture on a property sale. The skill derives the taxable/recapture split from the raw sale
  figures, prices what it can, and names the rest as a calculator limitation rather than folding it
  into an ordinary capital-gains figure.

## Install

Drop this plugin into Claude Code or Cowork. Each skill loads automatically when an advisor's
request matches its trigger phrases (see each `SKILL.md` for the specifics).
