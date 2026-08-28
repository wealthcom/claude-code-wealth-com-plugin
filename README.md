# Wealth.com Plugin for Claude Code and Cowork

This repository provides an official Claude Code and Cowork plugin that bundles:

- **Six Skills** for financial advisors working estate planning, balance sheet, and tax cases
- The **Wealth.com MCP Server**, which gives Claude secure access to a household's live client data — estate documents, balance sheet, and filed tax returns

This plugin allows Claude Code and Cowork users to install everything — Skills + MCP server — with **one click**.

---

## 🚀 Features

### ✅ Fully packaged estate, balance sheet, and tax Skills

Six Skills for advisors working a household's file:

**Estate**

| Skill | Description |
|-------|-------------|
| `wealth-estate-client-brief` | Orient on a household: who the client is, and what's on file for them |
| `wealth-estate-document-deep-dive` | Go deep on one estate document: its executive summary, disposition diagram, flagged observations, and follow-up questions about its own terms |
| `wealth-estate-document-intake` | File a new document to a client's record and have it read |

**Balance sheet**

| Skill | Description |
|-------|-------------|
| `wealth-balance-sheet-review` | What a household owns, who owns it as recorded, where it's concentrated, and what the beneficiary designations say |

**Tax**

| Skill | Description |
|-------|-------------|
| `wealth-tax-executive-summary` | A descriptive overview of a client's filed tax return: household and filing, income, deductions, tax and rates, capital-gains and IRMAA positioning, payments and safe harbor, and what's in the return |
| `wealth-tax-scenarios` | Build and model a tax position for a year that hasn't been filed, either from the advisor's own assumptions or by rolling a filed return forward with changes |

Each skill ships its own `reference/` (domain rules and edge cases), `examples/` (an end-to-end transcript), and `evaluations/` (triggering, functional, and should-not-trigger test cases).

### ✅ Integrated Wealth.com MCP Server

Claude automatically connects to Wealth's hosted MCP server at:

```
https://mcp.wealth.com/mcp
```

This provides Claude with tools to:

- Look up a client and their household
- Read a client's document inventory and estate documents
- Read a household's balance sheet and beneficiary designations
- Read a client's filed federal and state tax returns
- Model a tax scenario for a year that hasn't been filed
- Upload and extract a new estate document

---

## 📦 Installation

### Claude Code

**1. Add this plugin's marketplace**

```bash
/plugin marketplace add wealthcom/claude-code-wealth-com-plugin
```

**2. Install the plugin**

```bash
/plugin install wealth-com-claude-code-plugin@wealth-com-claude-code-plugin-marketplace
```

**3. Restart Claude Code**

This ensures the MCP server starts correctly.

### Claude Cowork

Add this repository as a marketplace, then install `wealth-com-claude-code-plugin` from it.

---

## 🔑 Authentication

You'll need the `wealthcom-mcp` connector authorized for your account — the skills do nothing without it.

---

## 🙌 Credits

- **Skills** by Wealth.com
- **MCP Server** by Wealth.com
- **Plugin Specification** by Anthropic
