# Wayfront MCP Skills

AI skill files for the [Wayfront](https://wayfront.com) MCP server. These teach AI models (Claude, ChatGPT, OpenClaw, Cursor, etc.) how to discover and use Wayfront MCP tools without prior knowledge.

## Install via ClawHub

```bash
clawhub install wayfront
```

## Connection Setup

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
