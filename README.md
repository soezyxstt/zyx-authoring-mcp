# Zyx Authoring MCP plugin

This package connects an MCP host to the production Zyx Authoring MCP at:

`https://zyxacademy.com/api/mcp/authoring`

It includes five active authoring skills from this repository:

- `zyx-authoring-pipeline`
- `zyx-source-pack-mcp`
- `zyx-idea-bundle-mcp`
- `zyx-product-bundle-mcp`
- `zyx-question-authoring-mcp`

`zyx-authoring-pipeline` orchestrates the full `idea_product` state machine and delegates each authoring stage to the matching specialist skill. The four specialist skills remain independently invocable.

## Host manifests

- `.codex-plugin/plugin.json` is the Codex Desktop plugin manifest.
- `.claude-plugin/plugin.json` is the Claude Code Desktop plugin manifest.
- `.mcp.json` is shared by both plugin manifests.
- `claude-desktop-config.example.json` is a standalone MCP configuration template for the Claude chat tab.

## Authentication and scopes

The server does not use a static token in this package. It requires an active Zyx admin session. Hosts with MCP OAuth support should open the Zyx sign-in flow automatically.

- **Default OAuth scopes**: `authoring:read` and `authoring:stage`.
- **Supported explicit scopes**: `authoring:review` and `authoring:withdraw`.
- **Session connections**: An authenticated admin session connection receives all supported scopes.

A host without interactive OAuth can use a short-lived bearer token created by an authenticated admin at `/api/mcp/authoring/connection`. Never commit a connection token or put it in `.mcp.json`.

## Implemented tool surface

The Zyx Authoring MCP implements tools across workflow, validation, knowledge inspection, and lifecycle operations:

- **Workflow and catalog**: `workflow.list`, `catalog.list_courses`, `catalog.list_chapters`, `source.list_files`, `source.read_file`, `source.ingest`, `workflow.start`, `workflow.get_run`, `workflow.get_contract`.
- **Validation and staging**: Idea Bundle, Product Bundle, and Question Bank validation and staging tools.
- **Knowledge and analysis**: Idea, product, source, coverage, question-bank, and similarity inspection tools.
- **Lifecycle management**: staged-import inspection, restaging, review, question updates, Product Bundle withdrawal, and draft discard operations.

The MCP server remains authoritative for schema validation, provenance, quality gates, authentication, and staging.

## Safety boundaries

- There is no MCP publication tool; final publication remains in the Zyx admin application.
- Destructive lifecycle operations require explicit operator intent.
- Connection tokens and private credentials must never be committed to this repository.

## Codex Desktop

Use the package as a local Codex plugin source. The root manifest and `.mcp.json` provide the plugin metadata and remote MCP connection, while `skills/` contains the authoring workflows.

## Claude Code Desktop

Claude Code Desktop can load the `.claude-plugin` manifest, root `skills/` directory, and `.mcp.json`. Reload installed plugins after changing the package.

## ChatGPT and other MCP hosts

Connect the host to:

`https://zyxacademy.com/api/mcp/authoring`

Authenticate through the Zyx admin OAuth flow when supported. Hosts without interactive OAuth can use the short-lived connection-token flow provided by the application.

## Project context

This repository is the client/plugin surface for the Zyx Academy authoring system. Server-side validation, persistence, review, and publication logic remain in the main Zyx application rather than being duplicated here.
