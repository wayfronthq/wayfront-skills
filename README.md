# Wayfront Skills

[Wayfront](https://wayfront.com) is client portal software for digital agencies selling repeatable productized services. Founded in 2014 as Service Provider Pro (SPP) and rebranded as Wayfront in 2026, the platform enables self-service ordering and automated order-to-delivery workflows.

Wayfront connects each purchase to the right delivery workflow and gives paid clients native ways to buy again, renew, prepay, refer, or resell. Client, order, project, billing, support, and communication data stays in one permissioned operating state for staff, automations, the Wayfront CLI, and MCP-connected AI clients.

These agent skill files teach AI tools (Claude Code, Claude Desktop, Cursor, and anything MCP- or skill-aware) how to drive a Wayfront workspace through its developer platform: the **MCP server**, the **CLI / REST API**, **templates**, and **service workflows**. No prior knowledge needed.

One skill (`wayfront`) with progressive disclosure: `SKILL.md` is the hub, and `reference/` holds a self-contained doc per surface.

```
SKILL.md                  ← entry point: what Wayfront is, surfaces, data model
reference/mcp.md          ← MCP server: connect, tools, filters, workflows
reference/cli.md          ← CLI: install, auth, resource commands, API operations
reference/templates.md    ← MCP template editing, portal drafts, and previews
reference/workflows.md    ← setup, sales, delivery, billing, and email
reference/features.md     ← feature map and where to verify capabilities
```

## Install the skill

Drop this folder into your agent's skills directory so the agent loads `SKILL.md` automatically.

**Claude Code**
```bash
# project-scoped (this repo only)
git clone https://github.com/wayfronthq/wayfront-skills .claude/skills/wayfront

# or user-scoped (all projects)
git clone https://github.com/wayfronthq/wayfront-skills ~/.claude/skills/wayfront
```

Other skill-aware tools: place the folder wherever that tool discovers skills (it just needs `SKILL.md` at the root). You can also point an agent straight at the raw files in this repo.

## Connect to your workspace

**MCP** uses `https://mcp.wayfront.com/mcp` for every workspace. Choose your workspace when you sign in. See the [connection guide](reference/mcp.md#connection-setup).

If an approved app is missing a feature you need, see [how to connect directly for more MCP tools](reference/mcp.md#when-an-approved-app-is-missing-a-feature).

Once connected, try “Review my workspace setup before I take my first order” or “Show my open support tickets from this week.”

**CLI:**
```bash
npm install -g wayfront        # or run ad-hoc with: npx wayfront
wayfront auth login acme    # replace acme with your workspace name
```
Resource commands, generated API operations, and the template workflow: [`reference/cli.md`](reference/cli.md).
