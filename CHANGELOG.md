# Changelog

All notable changes to this plugin are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html). The top version here is kept in lockstep
with the `version` field in `.claude-plugin/plugin.json`, and each release is tagged `vX.Y.Z`.

## [Unreleased]

### Added

- **`wealth-tax-insights`** — a new tax Skill that runs a structured review pass over a filed
  return and surfaces the gaps and planning opportunities worth a client conversation: anomalies
  where the return doesn't tie to itself, deductions and elections that are expected but absent,
  thresholds the household sits near, composition signals, and life-event inferences — each item
  carrying the mechanism behind it so the advisor can confirm it against the return. Ships
  `SKILL.md`, `references/knowledge-base.md` (detection modes, suppression rules, and the
  federal/state constants-tool contracts), and `references/example-riley.md` (the reference output
  to match for shape and tone). Descriptive overviews stay in `wealth-tax-executive-summary` and
  forward-looking modeling in `wealth-tax-scenarios`.

### Changed

- Revised the two tax Skills:
  - **`wealth-tax-scenarios`** — rewrote `SKILL.md` around an explicit path-selection flow (assumptions
    vs. rolling a filed return forward), a tiered derivation (landmarks → offered line-by-line
    worksheet), and a hardened constraint list; replaced the supporting docs with
    `references/knowledge-base.md` (tool contracts).
  - **`wealth-tax-executive-summary`** — rewrote `SKILL.md` and replaced the supporting docs with
    `references/knowledge-base.md` (federal/state tax-constant contracts and graceful degradation) and
    `references/example-riley.md` (the reference output to match for shape and tone).

## [0.4.0] - 2026-09-02

### Added

- Six Skills for financial advisors:
  - **Estate** — `wealth-estate-client-brief`, `wealth-estate-document-deep-dive`,
    `wealth-estate-document-intake`
  - **Balance sheet** — `wealth-balance-sheet-review`
  - **Tax** — `wealth-tax-executive-summary`, `wealth-tax-scenarios`
- Integrated `wealthcom-mcp` MCP server wiring (`.mcp.json`) for live client data — estate
  documents, balance sheets, and filed federal and state tax returns.
- One-click install via the plugin marketplace (`.claude-plugin/marketplace.json`).
- Trust & Security documentation ([SECURITY.md](SECURITY.md)) covering the OAuth 2.0 auth model,
  the data-flow boundary, and tenant/household scoping.
