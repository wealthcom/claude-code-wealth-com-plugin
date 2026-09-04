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

### 🔒 Data boundary

Because `wealthcom-mcp` is a remote, third-party MCP server, Claude Code and Cowork show a standing notice that they cannot verify servers they do not operate. That notice fires for **every** remote connector and is expected here — it is not a signal that anything is wrong with this plugin.

What that connection does, in plain terms:

- The connector authenticates over **OAuth 2.0** (`authenticate` / `complete_authentication`). Claude never sees or stores your Wealth.com password.
- Client records stay on Wealth.com until a skill asks for them. When Claude reads a household's estate documents, balance sheet, or tax return, **only the specific records your request needs** are returned into the session to answer that prompt.
- Requests are scoped to the households your Wealth.com account is entitled to; the connector does not expose other advisors' or other tenants' clients.

For the full request/response data flow, what stays on Wealth.com versus what enters the Claude session, and the tenant-scoping model, see **[SECURITY.md](./SECURITY.md)**.

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

**1. Add this plugin's marketplace**

```bash
/plugin marketplace add wealthcom/claude-code-wealth-com-plugin
```

**2. Install the plugin**

```bash
/plugin install wealth-com-claude-code-plugin@wealth-com-claude-code-plugin-marketplace
```

---

## 🔑 Authentication

The skills do nothing until the `wealthcom-mcp` connector is authorized for your account. Authorization uses **OAuth 2.0**: you sign in to Wealth.com in your browser, and Claude receives only a scoped access token — it never sees your Wealth.com username, password, or other credentials.

**1. Trigger the sign-in**

After restarting (see Installation above), run:

```bash
/mcp
```

Select `wealthcom-mcp` and choose to authenticate. (Invoking any Wealth.com skill also prompts the sign-in if the connector isn't yet authorized.) Your browser opens the Wealth.com sign-in page; complete it there and return to Claude.

**2. Check status**

Run `/mcp` at any time to see whether `wealthcom-mcp` is connected and authorized.

**3. Re-authorize when the session expires**

Authorization is time-limited. When your token expires, the skills stop returning data — run `/mcp`, select `wealthcom-mcp`, and authenticate again to restore access.

---

## 🔒 Trust & Security

This plugin's Skills call the hosted `wealthcom-mcp` connector, which serves live client data — estate documents, balance sheets, and filed tax returns.

- The connector authenticates over **OAuth 2.0**; you authorize it per account before any Skill can read data.
- Claude reaches the connector over HTTPS at `https://mcp.wealth.com/mcp`.
- This repository ships Skills and connector configuration only. It contains no client data and no credentials.

See [SECURITY.md](./SECURITY.md) for the data-flow diagram, OAuth scopes, data handling, retention, and compliance details.

---

## 🙌 Credits

- **Skills** by Wealth.com
- **MCP Server** by Wealth.com
- **Plugin Specification** by Anthropic
