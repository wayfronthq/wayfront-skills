# Wayfront Skills

[Wayfront](https://wayfront.com) is the AI-ready operating system for productized agencies: client billing, onboarding, and project delivery in one platform instead of five.

These agent skill files teach AI tools (Claude Code, Claude Desktop, Cursor, and anything MCP- or skill-aware) how to drive a Wayfront workspace through its developer platform: the **MCP server**, the **CLI / REST API**, **templates**, and **service workflows**. No prior knowledge needed.

One skill (`wayfront`) with progressive disclosure: `SKILL.md` is the hub, and `reference/` holds a self-contained doc per surface.

```
SKILL.md                  ← entry point: what Wayfront is, surfaces, data model
reference/mcp.md          ← MCP server: connect, tools, filters, workflows
reference/cli.md          ← CLI: install, auth, resource commands, API operations
reference/templates.md    ← portal/email Twig templates: pull/push/reset, naming
reference/workflows.md    ← how services, forms, orders & billing fit together (+ setup)
reference/features.md     ← full feature map: billing, support, portal, integrations, more
```

## Install the skill

Drop this folder into your agent's skills directory so the agent loads `SKILL.md` automatically.

**Claude Code / Claude Desktop**
```bash
# project-scoped (this repo only)
git clone https://github.com/wayfronthq/wayfront-skills .claude/skills/wayfront

# or user-scoped (all projects)
git clone https://github.com/wayfronthq/wayfront-skills ~/.claude/skills/wayfront
```

Other skill-aware tools: place the folder wherever that tool discovers skills (it just needs `SKILL.md` at the root). You can also point an agent straight at the raw files in this repo.

## Connect to your workspace

> Replace `[WORKSPACE]` with your workspace name. If you log in at `acme.wayfront.com`, your workspace is `acme`.

**MCP (Claude Code):**
```bash
claude mcp add --transport http wayfront https://[WORKSPACE].wayfront.com/mcp
```
The browser opens to authorize on first use. Claude Desktop and other MCP clients: see [`reference/mcp.md`](reference/mcp.md).

**CLI:**
```bash
npm install -g wayfront        # or run ad-hoc with: npx wayfront
wayfront auth login [WORKSPACE]
```
Resource commands, generated API operations, and the template workflow: [`reference/cli.md`](reference/cli.md).
