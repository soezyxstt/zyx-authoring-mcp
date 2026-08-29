# Zyx Authoring MCP

Plugin package for connecting MCP-capable hosts to the **Zyx Academy authoring system**.

The remote MCP endpoint is:

`https://zyxacademy.com/api/mcp/authoring`

## Included authoring skills

- `zyx-source-pack-mcp` — prepare and validate source material
- `zyx-idea-bundle-mcp` — author structured idea bundles
- `zyx-product-bundle-mcp` — generate learning products from published ideas
- `zyx-question-authoring-mcp` — author question-bank content

## Supported hosts

The repository contains manifests and examples for multiple MCP hosts:

- `.codex-plugin/plugin.json` — Codex plugin manifest
- `.claude-plugin/plugin.json` — Claude Code plugin manifest
- `.mcp.json` — shared MCP server configuration
- `claude-desktop-config.example.json` — standalone configuration example
- `skills/` — authoring workflow instructions

## Architecture

```text
MCP Host
  ├─ Codex
  ├─ Claude
  └─ other compatible clients
       │
       │ MCP over HTTP
       ▼
Zyx Authoring MCP
       │
       ├─ authentication
       ├─ workflow scope
       ├─ validation
       ├─ provenance checks
       ├─ quality gates
       └─ staging
       │
       ▼
Zyx Academy
```

The server remains authoritative for access control, schema validation, provenance, quality reports, and staging. The files in this repository provide host configuration and workflow guidance.

## Authentication

The plugin does **not** contain a permanent authentication token. The remote service requires an authenticated Zyx admin session.

Hosts with MCP OAuth support can use the interactive sign-in flow. Hosts without interactive OAuth may use a short-lived connection token created by an authenticated admin through the Zyx connection endpoint.

> Never commit connection tokens, session credentials, or other secrets to this repository or `.mcp.json`.

## Development

When changing a skill or host manifest:

1. Keep MCP endpoint configuration consistent across manifests.
2. Keep host-specific files thin; business rules belong on the server.
3. Update the relevant skill when the expected authoring workflow changes.
4. Validate the plugin before publishing or installing a new revision.

## Related project

This package is part of [Zyx Academy](https://zyxacademy.com), an AI-enabled learning platform with structured content authoring, assessment, retrieval, and tutoring workflows.
