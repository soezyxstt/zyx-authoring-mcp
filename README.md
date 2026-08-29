# Zyx Authoring MCP

Public plugin and client surface for the Zyx Academy authoring system.

**Endpoint:** `https://zyxacademy.com/api/mcp/authoring`

## Skills

- `zyx-authoring-pipeline`
- `zyx-source-pack-mcp`
- `zyx-idea-bundle-mcp`
- `zyx-product-bundle-mcp`
- `zyx-question-authoring-mcp`

The main pipeline coordinates the authoring workflow and delegates each stage to the matching specialist skill.

## Host files

- `.codex-plugin/plugin.json`: Codex plugin manifest
- `.claude-plugin/plugin.json`: Claude Code plugin manifest
- `.mcp.json`: shared MCP configuration
- `skills/`: authoring workflows

## Authentication

The server uses Zyx admin authentication rather than a static token stored in this repository.

Default scopes:

- `authoring:read`
- `authoring:stage`

Additional supported scopes:

- `authoring:review`
- `authoring:withdraw`

Never commit connection tokens or private credentials.

## Tool areas

- Workflow and catalog inspection
- Source ingestion
- Validation and staging
- Knowledge, coverage, and similarity analysis
- Review and lifecycle operations

Final publication remains in the Zyx admin application. The MCP server remains authoritative for validation, provenance, permissions, and staging.

## Usage

Connect an MCP-compatible host to:

`https://zyxacademy.com/api/mcp/authoring`

Use the Zyx admin OAuth flow when supported. Hosts without interactive OAuth can use the short-lived connection-token flow provided by the application.
