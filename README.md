# Since.dev MCP

Watch dependencies, verify facts, inspect coverage, and catch up on changes with durable cursors.

[Product](https://since.dev) · [Documentation](https://since.dev/docs)

Endpoint: `https://api.since.dev/mcp` (Streamable HTTP).

Connect using browser OAuth when your client supports it, or provide a workspace API key as a Bearer token. No tools run without authentication. Free credentials discover every tool family; workspace access and plan limits are enforced when a tool is called.

## Client configuration

Use [mcp.json](examples/mcp.json) for clients supporting environment expansion; replace the environment placeholder using your client's secret configuration if it does not. VS Code uses [vscode-mcp.json](examples/vscode-mcp.json). Never commit an API key.

For Gemini CLI, set SINCE_API_KEY in your shell and install this repository as an extension. Review the manifest before installation.

This repository contains connection metadata only. The server runs at Since.dev.

Generated from the product's distribution/metadata.json. Version 1.0.0.
