<img src="https://since.dev/icon-512.png" alt="Since.dev" width="48" height="48">

# Since.dev MCP

What changed in the packages, APIs and services your code depends on, and the fixes it needs.

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=since&config=eyJ1cmwiOiJodHRwczovL2FwaS5zaW5jZS5kZXYvbWNwIn0%3D)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=since&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.since.dev%2Fmcp%22%7D)

[Product](https://since.dev) · [Documentation](https://since.dev/docs) · [MCP reference](https://since.dev/docs#mcp)

Since.dev tracks every package in 18 registries, every AI model's retirement on each platform that serves it, API changes and runtime end-of-life dates, each with its source. This repository connects your agent to it: a hosted remote MCP server, plus a plugin for Claude Code, Cursor and Codex that tells the agent when to use it. There is no server code to install or run.

## Install

Every install signs in through your browser (OAuth). You approve the client for one Since.dev workspace, and no key is stored in a file. A free account works.

### Claude (claude.ai and the Claude apps)

Open [Since.dev in the connector directory](https://claude.ai/directory/connectors/since-dev), click Connect and sign in.

### Claude Code

Install the plugin, which adds the server and four skills:

```text
/plugin marketplace add since-dev/mcp
/plugin install since-dev@since-dev
```

Or add only the server, then run `/mcp` to sign in:

```sh
claude mcp add --transport http since https://api.since.dev/mcp
```

### Cursor

Click **Add to Cursor** above, or install the Cursor plugin ([Since-dev/cursor](https://github.com/since-dev/cursor)) to get the server, the skills and a rule that checks dependency, model and runtime changes.

### VS Code

Click **Install in VS Code** above, or run:

```sh
code --add-mcp '{"name":"since","type":"http","url":"https://api.since.dev/mcp"}'
```

### Codex

Install the plugin:

```sh
codex plugin marketplace add since-dev/mcp
```

Or add only the server and sign in:

```sh
codex mcp add since --url https://api.since.dev/mcp
codex mcp login since
```

### Gemini CLI

```sh
gemini extensions install https://github.com/since-dev/mcp
```

Then sign in through the browser from Gemini CLI with `/mcp auth since`. Or add only the server:

```sh
gemini mcp add --transport http since https://api.since.dev/mcp
```

### Any other MCP client

Add a remote server with this URL over Streamable HTTP and sign in when the client asks:

```text
https://api.since.dev/mcp
```

For a client that reads an `mcpServers` file, see [examples/mcp.json](examples/mcp.json). For VS Code's `.vscode/mcp.json`, see [examples/vscode-mcp.json](examples/vscode-mcp.json).

## Use an API key instead

Some clients can't sign in through a browser. For those, create a key and send it as a Bearer token:

1. Sign in at https://app.since.dev and pick the workspace the agent should use.
2. Open **API keys** (https://app.since.dev/api-keys) and choose **Create API key**. The key starts with `sc_live_` and is shown once.
3. Add the header `Authorization: Bearer <your API key>` in your client's secret settings. Never commit a key.

A key reaches only its own workspace, over the API and MCP, and any member with a seat can revoke it under **API keys**.

## What's inside

- **The server:** 3 tools, from the same record as the Since.dev API. `whats_changed` and `check_code` only read, so your client runs them without asking. `fix_code` opens fix pull requests, so your client asks before every call.
- **Skills** (Claude Code, Cursor, Codex and other clients that read Agent Skills): [`since-before-upgrading`](skills/since-before-upgrading/SKILL.md), [`since-model-retirements`](skills/since-model-retirements/SKILL.md), [`since-what-changed`](skills/since-what-changed/SKILL.md), [`since-ci-and-runtimes`](skills/since-ci-and-runtimes/SKILL.md). They tell the agent when to check: before upgrades, when code names a model, before writing against something it learned in training, and when editing CI or runtimes.
- **For Cursor:** the plugin at [Since-dev/cursor](https://github.com/since-dev/cursor) adds a rule that brings the same checks to manifest, lockfile, Dockerfile and workflow edits.

## Tools

`tools/list` returns each tool's full input schema.

### Tools

whats_changed and check_code only read, and both work on every plan; checking a connected repository (`check_code` with `repo`) is on the Business and Platform plans. fix_code writes: with `code` it returns a patch, on the Team plan and up, and with `repo` it opens fix pull requests for a person to review, on the Business and Platform plans. Each call costs credits; a bad input or a call the plan does not include is refused with a plain sentence and costs nothing.

| Tool | What it does | Key inputs |
| --- | --- | --- |
| `whats_changed` | What changed in a package, a provider, a model, a runtime or a connected repository, since your training cutoff, a date or a version, with severity, dates, replacements and cited evidence. Also lists a provider's current models to use in new code (current=true), gives one change in full with its migration steps (id), answers whether something is still supported, and lists upcoming retirements (as iCalendar too). | One of `package` (any spelling or purl), `provider`, `model`, `repo`, `surface` or `id` (one change, as a `check_code` finding names it); optional `since` (a date, a version or `cutoff:<model>`), `current` (with `provider`), `within` (for example `90d`), `kind`, `severity`, `reaches` (with `repo`), `cursor`, `limit` |
| `check_code` | Checks code against what changed, before you rely on it: a snippet you just wrote, a model id, a package version, a manifest or a whole repository. Returns what is affected, where (file and line), how severe, how many days are left, the replacement and the source. Anything Since cannot settle is `needs_review`, never a guess. It only reads. | One of `model`, `package` (with a version), `surface`, `code` (with `language`), `manifest` or `lockfile` (`path` and `content`), `repo`, or `repoUrl` (a public GitHub repository); optional `state`, `severity`, `detail` (`summary` or `full`), `limit`, `cursor` |
| `fix_code` | Fixes what check_code found: the patch for a snippet, in seconds, or a connected repository's fix pull requests, one per provider, each checked before it opens. Nothing merges: every pull request waits for a person to review it. Fixes count against the month's fix pull requests, not credits. | One of `repo` or `code` (with `language` or `path`); optional `state`, `severity`, `detail`, `limit` |

## Examples

Each step shows the `name` and `arguments` the agent sends in a `tools/call` request.

1. Before writing code against a model, check the id:

   ```json
   { "name": "check_code", "arguments": { "model": "<model id>" } }
   ```

   The result lists the model's status (`supported`, `deprecated`, `retired` or `unknown`) and a finding for each change that applies, with its severity, `daysLeft`, `replacement` and cited `source`.

2. Before upgrading a package, read what changed from the version you have:

   ```json
   { "name": "whats_changed", "arguments": { "package": "npm:react", "since": "18.2.0" } }
   ```

   Each change carries its canonical surface, kind, severity, dates, replacement and evidence. `package.releasesBehind` says how far behind the version is, and `coverage` says which sources were read and how fresh they are. Page with `nextCursor`.

3. See what retires in the next 90 days for a provider:

   ```json
   { "name": "whats_changed", "arguments": { "provider": "anthropic", "within": "90d" } }
   ```

4. Fix what a connected repository's check found (your client asks first):

   ```json
   { "name": "fix_code", "arguments": { "repo": "<repository id>" } }
   ```

Treat `unknown` and `needs_review` as not settled, never as fine.

## Requirements

- **A Since.dev account.** [Create one](https://app.since.dev/login?mode=signup) with email, GitHub or Google. Each account includes one Free workspace.
- **Credits.** Each call costs credits from the workspace's monthly allowance, and every response says how many are left. See [pricing](https://since.dev/#pricing).

## Support

- Documentation: [https://since.dev/docs](https://since.dev/docs), including the [MCP reference](https://since.dev/docs#mcp)
- Questions about this connection: [open an issue](https://github.com/since-dev/mcp/issues)
- Email: [support@since.dev](mailto:support@since.dev)

## License

The files in this repository are MIT licensed. See [LICENSE](LICENSE).
