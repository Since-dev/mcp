# Install the Since.dev MCP server

These steps are for an AI agent setting up Since.dev for its user.

Since.dev is a hosted remote MCP server over Streamable HTTP. There is nothing to download, build or run.

- **Server URL:** `https://api.since.dev/mcp`
- **Transport:** Streamable HTTP
- **Sign-in:** OAuth in the browser (preferred), or a workspace API key sent as `Authorization: Bearer <key>`

## Steps

1. Add the server to the client's MCP settings with the URL above and no headers. For a client that reads an `mcpServers` file:

   ```json
   {
     "mcpServers": {
       "since": {
         "url": "https://api.since.dev/mcp"
       }
     }
   }
   ```

2. When the client connects, it gets a sign-in link. Ask the user to open it, sign in or create a free Since.dev account, and approve the client for one workspace. Do not ask the user to paste a password into the chat.
3. If the client cannot sign in through a browser, ask the user to create an API key at https://app.since.dev/api-keys and add it as a header in the client's secret settings:

   ```json
   { "mcpServers": { "since": { "url": "https://api.since.dev/mcp", "headers": { "Authorization": "Bearer <the user's API key>" } } } }
   ```

   Never write the key into a file in the repository.
4. Confirm it works: list the tools. You should see `whats_changed`, `check_code` and `fix_code`.
5. Try one call, for example `check_code` with `{ "model": "gpt-4o-mini-2024-07-18" }`, and show the user the answer.

## Using it well

- Call `check_code` before upgrading a dependency, when code names an AI model, and when editing CI or runtime versions.
- Call `whats_changed` with `since` set to your training cutoff before writing code against a package or API you learned in training.
- `fix_code` writes: it opens fix pull requests for a person to review, so it always asks the user first.
- Treat `needs_review` and `unknown` as not settled, never as safe.

Documentation: https://since.dev/docs#mcp
