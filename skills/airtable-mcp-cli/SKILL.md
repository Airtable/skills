---
name: airtable-mcp-cli
description: How to use the Airtable MCP CLI to manage bases, tables, and records from the terminal. Use when the user wants to interact with Airtable from the command line or a script.
license: MIT
metadata:
    version: '1.0.0'
    author: airtable
---

# Airtable MCP CLI

The [Airtable MCP CLI](https://github.com/Airtable/airtable-mcp-cli) lets you manage your Airtable bases from the terminal. It discovers available tools from the Airtable MCP server at runtime, so you always have access to the latest capabilities without updating the CLI itself. Install it with `npm install -g @airtable/mcp-cli`, then run `airtable-mcp configure` to set up your [personal access token](https://airtable.com/create/tokens). From there, `airtable-mcp tools` lists available tools and `airtable-mcp <tool> --help` shows the flags for any specific tool.

For scripting and agent use, set `AIRTABLE_TOKEN` as an environment variable to skip interactive configuration. Use `--input -` to pass arguments as JSON on stdin, `-q` to suppress status messages, and `tools --json` to discover tools programmatically. Tool names and schemas come from the server and may change between server releases, so scripts should check for tool existence before calling.
