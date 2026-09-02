# Changelog

All notable changes to this plugin are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html). The top version here is kept in lockstep
with the `version` field in `.claude-plugin/plugin.json`, and each release is tagged `vX.Y.Z`.

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
