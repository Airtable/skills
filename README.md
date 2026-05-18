# Airtable

Airtable is the database and operations layer for your agents — whether running product, marketing, sales, ops, HR, or a custom business app. It combines structured data with multiplayer visual surfaces (grid, kanban, calendar, gallery, timeline) humans and agents share — plus sync integrations to Jira, Salesforce, Zendesk, Google Drive, Databricks, and the rest of your stack, all backed by enterprise governance.

This repo is the canonical, official home for the plugins, skills, and other artifacts we ship to make Airtable as fluent to agents as it already is to builders and users. Today that ships as an installable plugin for [Claude Code](https://code.claude.com) and [Codex](https://developers.openai.com/codex), and as a standalone collection of skills installable via the [`skills` CLI](https://github.com/vercel-labs/skills) for any agent that consumes [agentskills.io](https://agentskills.io)–format skills.

## What's inside

The bundled skills live under [`plugins/airtable/skills/`](./plugins/airtable/skills/), and the official Airtable MCP server (`mcp.airtable.com/mcp`) is auto-wired via `.mcp.json`.

## Install

### Claude Code

```bash
/plugin marketplace add airtable/skills
/plugin install airtable@airtable-skills
```

For local testing from a clone:

```bash
/plugin marketplace add /path/to/this/repo
/plugin install airtable@airtable-skills
```

### Codex

```bash
codex plugin marketplace add airtable/skills
```

Then enable the plugin via the Codex TUI's plugins menu, or by adding to `~/.codex/config.toml`:

```toml
[plugins."airtable@airtable-skills"]
enabled = true
```

If you'd rather wire the MCP server manually instead of via the plugin, add the following to `~/.codex/config.toml` (the plugin handles this automatically when enabled — this is just an escape hatch):

```toml
[[mcp_servers]]
name = "airtable"
type = "http"
url = "https://mcp.airtable.com/mcp"
```

### Standalone skills (any agent)

For Cursor, OpenCode, Pi, Gemini CLI, or any other agent that consumes [Agent Skills](https://agentskills.io):

```bash
# All skills
npx skills add airtable/skills

# A specific skill
npx skills add airtable/skills --skill airtable-filters
```

Or download the latest tagged release zip from [GitHub Releases](https://github.com/airtable/skills/releases/latest).

## Layout

```
.
├── .claude-plugin/marketplace.json     # Claude marketplace catalog
├── .agents/plugins/marketplace.json    # Codex marketplace catalog
└── plugins/airtable/                   # the V1 plugin
    ├── .claude-plugin/plugin.json
    ├── .codex-plugin/plugin.json
    ├── .mcp.json
    └── skills/
```

## License

[MIT](./LICENSE.md).
