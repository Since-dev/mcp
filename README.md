<img src="https://since.dev/icon-512.png" alt="Since.dev" width="48" height="48">

# Since.dev MCP

Grant, your COO, and his AI team: live answers, team status and decisions, plus World State.

[Product](https://since.dev) · [Documentation](https://since.dev/docs) · [MCP reference](https://since.dev/docs#mcp)

Since.dev is a hosted service, and this is its remote MCP server. It gives an AI agent three kinds of tools:

- **Grant:** see what Grant, your AI COO, and his team are working on, read findings and runs, ask Grant a question, stop running work and record decisions on findings.
- **World State:** watch the public things your code depends on (packages, repositories, service status pages, web pages and advisory catalogs), catch up on every change since the agent last looked using a durable cursor, and check remembered facts before acting on them.
- **Repositories and repairs:** read the repositories shared with the agent, their dependency impact and proposed repairs.

This repository holds connection files only: this README, client configuration examples and a Gemini CLI extension. There is no server code to install or run.

## Contents

- [Requirements](#requirements)
- [Get an API key](#get-an-api-key)
- [Connect](#connect)
- [Grant access](#grant-access)
- [Tools](#tools)
- [Example: watch a dependency and catch up on changes](#example-watch-a-dependency-and-catch-up-on-changes)
- [Support](#support)

## Requirements

- **A Since.dev account.** [Create one](https://app.since.dev/login?mode=signup) with email, GitHub or Google. Each login includes one Free workspace.
- **A credential for that workspace:** an API key, or browser sign-in (OAuth) in clients that support it. The server has no anonymous access, and a credential only reaches its own workspace.
- **A plan that includes the tools you use.** World State, repository and usage tools work on every plan, including Grant Free, within the plan's limits; call `get_usage` to see them. Grant tools need a paid plan (Grant Pro or Grant Studio); on Grant Free they return `plan_required`. See [pricing](https://since.dev/#pricing).

## Get an API key

1. Sign in at https://app.since.dev and select the workspace the agent should use.
2. Open **Connections → External agents** (https://app.since.dev/connections/agents).
3. Choose **New connection**, enter a connection name such as `release-bot`, then choose **Create connection**.
4. Copy the key. It starts with `sc_live_` and is shown only once. If you lose it, issue a new key from the connection's **Manage**.
5. Save it as the `SINCE_API_KEY` environment variable or in your client's secret settings. Never commit an API key.

Any workspace member can create a connection. Each connection is a separate agent in the workspace with its own keys. It reads changes for the watches it creates with `subscribe`; to give it existing watches, choose **Connect watches** on the connection or call `attach_to_set`. Repository access and [Grant access](#grant-access) are granted separately.

To skip the key, use browser sign-in: clients that support remote MCP sign-in, such as Claude, Codex and Cursor, can add the server URL and sign in when prompted. A workspace owner or admin approves the workspace, projects and permissions during sign-in.

## Connect

Add the server to your MCP client using Streamable HTTP:

```text
https://api.since.dev/mcp
```

- **OAuth:** sign in through your browser when your client supports it.
- **API key:** send your workspace API key as a Bearer token in the `Authorization` header.

### Clients that use an mcpServers file

From [examples/mcp.json](examples/mcp.json):

```json
{
  "mcpServers": {
    "since": {
      "url": "https://api.since.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${SINCE_API_KEY}"
      }
    }
  }
}
```

This example reads `SINCE_API_KEY` from the environment. If your client does not expand environment variables, use its secret settings instead.

### VS Code

From [examples/vscode-mcp.json](examples/vscode-mcp.json), for your workspace's `.vscode/mcp.json`. VS Code prompts for the key and keeps it as a secret input:

```json
{
  "servers": {
    "since": {
      "type": "http",
      "url": "https://api.since.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${input:since-key}"
      }
    }
  },
  "inputs": [
    {
      "id": "since-key",
      "type": "promptString",
      "description": "Since.dev API key",
      "password": true
    }
  ]
}
```

### Gemini CLI

```sh
gemini extensions install https://github.com/since-dev/mcp
```

Enter your Since.dev workspace API key at the installer's secret prompt.

## Grant access

New credentials have no Grant access. A workspace owner or admin opens **Connections → External agents**, chooses **Manage** on the connection and, under **Grant access**, picks the projects shared with it and its allowed actions: read, question, investigate, decision and control.

- Call `grant_team_status` first. It shows your approved projects, permissions and Grant's team, or where an owner grants access.
- Leave out `applicationIds` to cover every approved project.
- `grant_question` waits up to 55 seconds. If Grant is still working, call it again with the same `requestKey` to keep waiting. Questions use the same workspace model and allowances as the app.
- World State and repository tools keep their own access grants and work without Grant access.

## Tools

The server offers 20 tools, and `tools/list` returns each one's full input schema. A connection that uses browser sign-in lists the Grant tools its approved permissions allow.

### Grant

Grant tools need a paid plan and Grant access for the connection (see [Grant access](#grant-access)). Each one takes its arguments in an `input` object, plus an optional `applicationIds` array that narrows the projects. Without `applicationIds`, a call covers every project approved for the connection. Nullable fields inside `input` are still required: send `null` when one does not apply.

| Tool | What it does | Key inputs |
| --- | --- | --- |
| `grant_team_status` | Call this first. Shows your approved projects, permissions and Grant's team: each teammate's role, sources, state, current work and latest brief. Pass `runIds` to read the status of specific runs instead. | `input.runIds`: up to 20 run ids, or `null` for the overview |
| `grant_read` | Read one saved finding (with evidence and decisions), run (with its brief) or conversation by id, or list a page of findings. | `input.type` (`finding`, `run`, `conversation` or `findings`), `input.id`, `input.cursor`, `input.severity`, `input.disposition` |
| `grant_question` | Ask Grant about your approved projects. He answers from saved findings and briefs and can consult his team live, which also needs the `investigate` permission. Waits up to `waitSeconds` for the answer; if Grant is still working, call again with the same `requestKey` to keep waiting. | `input.content` (up to 12,000 characters), `input.conversationId` (or `null` for a new conversation), `input.waitSeconds` (0 to 55), `requestKey` (8 to 160 characters) |
| `grant_stop` | Stop Grant's or a teammate's running work. Saved results are kept. | `input.runId` |
| `grant_record_decision` | Record a disposition, note, owner or follow-up date on a finding. This records a decision; it does not prove the issue is fixed. | `input.id`, `input.expectedVersion`, `input.disposition` (`open`, `accepted`, `deferred`, `addressed` or `dismissed`), `input.note`; optional `input.ownerId`, `input.revisitAt` |

### World State

World State tools work on every plan, within the plan's limits. A connection reads changes for the watches it creates and the watches or sets shared with it.

| Tool | What it does | Key inputs |
| --- | --- | --- |
| `check_coverage` | Ask whether something can be watched before you build on it. Returns the coverage level (`good`, `partial` or `none`), the sources, the check cadence and the canonical subject key. | `subjectType`, `subject`, `aspect` |
| `subscribe` | Create a watch so changes reach you without polling. Use the typed form for one watch, or a plain-language statement that names up to ten. | `subjectType`, `aspect` and one of `subject` or `canonicalKey`; or `statement`. Optional `threshold` (0 to 1) |
| `what_changed_since` | The first call when an agent resumes. Returns every change to your watches since your acknowledged cursor, ordered and deduplicated, each with field changes, source evidence and an assessment. | Optional `since` (a cursor; leave it out to resume from your acknowledged position), `limit` (default 50, max 200) |
| `acknowledge_changes` | Advance your durable cursor after you have processed a complete `what_changed_since` response. Safe to retry; the cursor never moves backwards. | `cursor` from that response |
| `verify_facts` | Check facts the agent remembers against current observed state before acting on them. Each fact returns `valid`, `changed` or `unknown`; `unknown` means there is no observation, not that nothing changed. Counts toward the plan's monthly verification calls. | `facts`: up to 100, each with `subjectKey`, `aspect`, `fieldPath`, `expectedValue` |
| `subject_history` | A subject's observed change timeline and totals, so the agent can choose a sensible re-check cadence. | `subjectKey`; optional `limit` |
| `list_watches` | List the saved watches this connection can see, 50 per page. | Optional `status` (`active`, `needs_attention`, `paused`, `closed` or `expired`), `offset` |
| `inspect_watch` | Read one saved watch with its coverage and source health. Does not check sources or acknowledge changes. | `id` |
| `list_sets` | List the workspace's subscription sets (named groups of watches) and whether this connection is attached to each. | None |
| `attach_to_set` | Attach to a subscription set so its changes reach you from now on. Earlier changes are not replayed; use `subject_history` for those. | `setId` from `list_sets` |

### Repositories and repairs

These tools read only repositories that a workspace owner or admin has shared with the connection, in the repository's **Settings → Agent access**. They grant no write access.

| Tool | What it does | Key inputs |
| --- | --- | --- |
| `list_maintained_repositories` | List the repositories shared with this connection. | None |
| `inspect_repository` | Read a saved repository with its dependency and impact status. | `id`; optional `impactOffset` |
| `list_repairs` | Read impact and repair status for shared repositories. | Optional `repositoryId`, `review` (`open`, `attention`, `completed` or `all`), `offset` |
| `inspect_repair` | Read one repair's captured evidence, affected code, patch versions, CI checks and decisions. A passing check is not proof of production safety. | `id`; optional `includeCurrentPr` to include the current GitHub pull request diff |

### Account

| Tool | What it does | Key inputs |
| --- | --- | --- |
| `get_usage` | Read the workspace's plan and current usage. Read-only. | None |

## Example: watch a dependency and catch up on changes

An agent that maintains a project using React wants to hear about new React versions without polling npm. Each step shows the `name` and `arguments` the agent sends in a `tools/call` request.

1. Watch the dependency. To check first whether a source is covered, call `check_coverage` with the same three fields.

   ```json
   { "name": "subscribe", "arguments": { "subjectType": "package", "subject": "npm:react", "aspect": "versions" } }
   ```

   The result is the new watch with its `id`, `status`, `coverage` and `cadenceMinutes`. The first check records a baseline, and later changes are delivered to this connection.

2. When the agent resumes work, call `what_changed_since` before anything else. Leave out `since` to continue from the last acknowledged position.

   ```json
   { "name": "what_changed_since", "arguments": { "limit": 50 } }
   ```

   Abridged example result:

   ```text
   {
     "changes": [
       {
         "change": {
           "seq": 84219,
           "idempotencyKey": "a3f1c9...",
           "subject": { "key": "package:npm:react", "type": "package", "name": "react" },
           "aspect": "versions",
           "fieldChanges": [{ "path": "version", "label": "Latest version", "from": "19.1.0", "to": "19.2.0" }],
           ...
         },
         "context": { ... }
       }
     ],
     "cursor": 84219,
     "hasMore": false,
     ...
   }
   ```

   Each `change` block holds the field changes, source and detection time. Its `context` holds the evidence and assessment.

3. After processing every change in the response, acknowledge its cursor:

   ```json
   { "name": "acknowledge_changes", "arguments": { "cursor": 84219 } }
   ```

   If `hasMore` was true, call `what_changed_since` again. A page that was never acknowledged is returned again, so delivery is at least once: deduplicate on `idempotencyKey`.

4. Before acting on a version the agent remembered, check it:

   ```json
   {
     "name": "verify_facts",
     "arguments": {
       "facts": [
         { "subjectKey": "package:npm:react", "aspect": "versions", "fieldPath": "version", "expectedValue": "19.2.0" }
       ]
     }
   }
   ```

   Each fact returns `valid`, `changed` or `unknown`. Treat `unknown` as "no observation", never as "unchanged".

## Support

- Documentation: [https://since.dev/docs](https://since.dev/docs), including the [MCP reference](https://since.dev/docs#mcp)
- Questions about this connection: [open an issue](https://github.com/since-dev/mcp/issues)
- Email: [support@since.dev](mailto:support@since.dev)
