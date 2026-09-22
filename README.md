<img src="https://since.dev/icon-512.png" alt="Since.dev" width="48" height="48">

# Since.dev MCP

Watch dependencies, verify facts, inspect coverage, and catch up on changes with durable cursors.

[Product](https://since.dev) · [Documentation](https://since.dev/docs)

## Connect

Add this remote server in your MCP client using Streamable HTTP:

```text
https://api.since.dev/mcp
```

- **OAuth:** sign in through your browser when your client supports it.
- **API key:** provide your workspace API key as a Bearer token.

Configuration examples: [VS Code](examples/vscode-mcp.json) · [Other clients](examples/mcp.json).

The generic example uses `SINCE_API_KEY`. Use your client's secret settings if it does not expand environment variables. Never commit an API key.

## Gemini CLI

```sh
gemini extensions install https://github.com/since-dev/mcp
```

Enter your Since.dev workspace API key at the installer's secret prompt.
