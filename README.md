# Wayfront Skills

AI skill files for working with [Wayfront](https://wayfront.com). These teach AI models (Claude, ChatGPT, OpenClaw, Cursor, etc.) how to use a Wayfront workspace — across the **MCP server**, the **CLI / REST API**, **templates**, and **services** — without prior knowledge.

This is one installable skill (`wayfront`) using progressive disclosure: `SKILL.md` is the hub, and `reference/` holds a self-contained doc per surface.

```
SKILL.md                  ← entry point: what Wayfront is, the 3 surfaces, data model
reference/mcp.md          ← MCP server: connect, tools, Purity filters, workflows
reference/cli.md          ← CLI: install, auth, resource commands, raw API
reference/templates.md    ← portal/email Twig templates: pull/push/reset, naming
reference/services.md     ← the product catalogue: fields, billing, setup
```

## Install via ClawHub

```bash
clawhub install wayfront
```

## MCP Connection Setup

Wayfront's MCP server supports OAuth 2.1 with dynamic client registration — clients that support MCP OAuth (like Claude Code and Claude Desktop) handle authentication automatically. You just need your workspace URL.

> Replace `[WORKSPACE]` below with your Wayfront workspace name — for example, if you log in at `acme.wayfront.com`, your workspace is `acme`.

### Claude Code

```bash
claude mcp add --transport http wayfront https://[WORKSPACE].wayfront.com/mcp
```

Claude Code will open your browser to authorize when you first use a Wayfront tool.

### Claude Desktop

1. Open **Settings → MCP Servers → Add**
2. Enter `https://[WORKSPACE].wayfront.com/mcp` as the server URL
3. You'll be prompted to authorize in your browser

### Other MCP Clients

Any MCP client that supports Streamable HTTP + OAuth can connect:

- **Endpoint:** `https://[WORKSPACE].wayfront.com/mcp`
- **Transport:** Streamable HTTP
- **Auth:** OAuth 2.1 (discovery via `/.well-known/oauth-authorization-server`)

## CLI Setup

```bash
npm install -g wayfront        # or run ad-hoc with: npx wayfront
wayfront auth login [WORKSPACE]
```

See [`reference/cli.md`](reference/cli.md) for resource commands, the raw API escape hatch, and the template workflow.
