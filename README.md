# Arkanva MCP server

[![npm](https://img.shields.io/npm/v/@arkanva/mcp?color=8E7AB5&label=npm)](https://www.npmjs.com/package/@arkanva/mcp)
[![node](https://img.shields.io/node/v/@arkanva/mcp?color=8E7AB5)](https://nodejs.org)
[![license](https://img.shields.io/npm/l/@arkanva/mcp?color=2E6F40)](./LICENSE)
[![MCP](https://img.shields.io/badge/MCP-stdio-8E7AB5)](https://modelcontextprotocol.io)

The official [Model Context Protocol](https://modelcontextprotocol.io) server for
[Arkanva](https://arkanva.com), the hand-drawn 3D pastel diagramming workspace. It lets Claude
Code, Claude Desktop, Cursor, Windsurf, Codex and any other MCP client create, read, edit,
import, export and share Arkanva architecture diagrams, flowcharts and system designs in your
workspace.

- 20 tools: diagrams, folders, edits with auto-layout, symbol search across AWS, Azure, Google
  Cloud, Alibaba Cloud and generic catalogues, templates, Mermaid / draw.io / Excalidraw import,
  SVG / PNG / GIF / Mermaid / draw.io export, sensitive-data masking and share links.
- Atomic edits: every batch of up to 200 operations applies completely or not at all, checked
  against the diagram revision.
- Human in the loop: deletions, large edits and public sharing come back as
  `confirmation_required` and only apply after you approve them.
- A single self-contained file with no runtime dependencies. It talks to Arkanva over HTTPS
  with your agent token and writes nothing but MCP frames to stdout.

## Requirements

- Node.js 20 or newer.
- An Arkanva account on a plan that includes agent access.
- An agent token (`akf_…`): open [arkanva.com/app](https://arkanva.com/app), then
  **Account → Agent access → Create token**. The token is shown once; copy it.

## Install

You do not install anything by hand: your MCP client runs `npx -y @arkanva/mcp`, which downloads
and starts the latest release. To pin it globally instead, run `npm install -g @arkanva/mcp` and
use `arkanva-mcp` as the command.

### Claude Code

```bash
claude mcp add -s user arkanva --env ARKANVA_API_KEY=akf_your_token -- npx -y @arkanva/mcp
```

`-s user` makes the server available in every project. Check it with `claude mcp list`.

### Claude Desktop

Edit `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/`, Windows:
`%APPDATA%\Claude\`) and restart the app:

```json
{
  "mcpServers": {
    "arkanva": {
      "command": "npx",
      "args": ["-y", "@arkanva/mcp"],
      "env": { "ARKANVA_API_KEY": "akf_your_token" }
    }
  }
}
```

If Claude Desktop on Windows cannot start `npx`, use `"command": "cmd"` with
`"args": ["/c", "npx", "-y", "@arkanva/mcp"]`.

### Cursor

Add the same `mcpServers` block to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one
project; keep the token out of version control).

### Windsurf

Add the same `mcpServers` block to `~/.codeium/windsurf/mcp_config.json`, then refresh the MCP
servers in Windsurf settings.

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "arkanva": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@arkanva/mcp"],
      "env": { "ARKANVA_API_KEY": "akf_your_token" }
    }
  }
}
```

### Codex

`~/.codex/config.toml`:

```toml
[mcp_servers.arkanva]
command = "npx"
args = ["-y", "@arkanva/mcp"]
env = { ARKANVA_API_KEY = "akf_your_token" }
```

## Configuration

| Variable              | Required | Default               | Purpose                                                                      |
| --------------------- | -------- | --------------------- | ---------------------------------------------------------------------------- |
| `ARKANVA_API_KEY`     | yes      |                       | Agent token (`akf_…`). `ARKANVA_TOKEN` is accepted as an alias.              |
| `ARKANVA_API_URL`     | no       | `https://arkanva.com` | Arkanva address, for self-hosted servers. Must be https, except `localhost`. |
| `ARKANVA_CHROME_PATH` | no       | auto-detected         | Chrome, Chromium or Edge used to render PNG and GIF exports.                 |

```bash
npx -y @arkanva/mcp --help
npx -y @arkanva/mcp --version
```

The token sits in your client configuration in plain text. Keep that file private, never commit
it, and revoke the token under Account → Agent access if it leaks. Tokens expire; when one does,
tools answer with a `401` error that tells you to create a new one.

## Tools

| Tool                           | What it does                                                          | Access      |
| ------------------------------ | --------------------------------------------------------------------- | ----------- |
| `arkanva_describe`             | API version, token scopes, plan features and limits                   | read-only   |
| `arkanva_list_documents`       | List diagrams, optionally in one folder                               | read-only   |
| `arkanva_open_document`        | Full diagram content and revision                                     | read-only   |
| `arkanva_get_element`          | One node or edge                                                      | read-only   |
| `arkanva_create_document`      | Create an empty diagram                                               | additive    |
| `arkanva_update_document`      | Rename a diagram or move it between folders                           | additive    |
| `arkanva_apply_operations`     | Add, update, remove and reorder nodes and edges, with optional layout | destructive |
| `arkanva_delete_document`      | Delete a diagram (asks for confirmation)                              | destructive |
| `arkanva_search_symbols`       | Search cloud, infrastructure, UML, flowchart and other symbols        | read-only   |
| `arkanva_list_style_tokens`    | Pastel colour and style tokens for nodes and edges                    | read-only   |
| `arkanva_list_folders`         | List folders                                                          | read-only   |
| `arkanva_create_folder`        | Create a folder                                                       | additive    |
| `arkanva_update_folder`        | Rename a folder                                                       | additive    |
| `arkanva_delete_folder`        | Delete a folder, keeping or deleting its diagrams (asks first)        | destructive |
| `arkanva_import_document`      | New diagram from Mermaid, draw.io, Excalidraw or `.arkanva.json`      | additive    |
| `arkanva_export_document`      | SVG, PNG, animated GIF, Mermaid, draw.io or `.arkanva.json`           | additive    |
| `arkanva_list_templates`       | Official templates                                                    | read-only   |
| `arkanva_create_from_template` | New diagram from a template                                           | additive    |
| `arkanva_sanitize_document`    | Find and mask IPs, hosts, emails, keys and ARNs                       | destructive |
| `arkanva_manage_share`         | Public read-only link status, enable (asks first), disable, refresh   | additive    |

Every input is validated against its JSON schema before any request is sent. Full parameter
reference: [arkanva.com/docs](https://arkanva.com/docs).

## Example prompts

- "Draw the architecture of a serverless checkout on AWS in Arkanva: API Gateway, Lambda,
  DynamoDB and SQS, laid out left to right."
- "Open my _Payments_ diagram and add a Redis cache between the API and Postgres."
- "Import this Mermaid flowchart into Arkanva in the _Onboarding_ folder."
- "Export the _Platform overview_ diagram as a PNG at 2x to ./platform.png."
- "Check the _Network_ diagram for IP addresses and secrets and show me what would be masked."

## Errors

Failures come back as MCP tool errors (`isError: true`) with the HTTP status, an error `code` and
a hint. The common ones:

| Code                    | Meaning                                                             |
| ----------------------- | ------------------------------------------------------------------- |
| `unauthorized` (401)    | Token wrong, expired or revoked: create a new one                   |
| `insufficient_scope`    | Token lacks the scope, e.g. `documents:delete`                      |
| `plan_required`         | The feature is not in the account plan                              |
| `quota_exceeded`        | Plan limit reached; the error includes `limit` and `current`        |
| `rate_limited` (429)    | Too many calls; wait a minute                                       |
| `revision_conflict`     | The diagram changed; open it again and rebuild the batch            |
| `confirmation_required` | Ask the user, then repeat the call with the returned `confirmation` |

Startup problems (missing token, bad URL) are printed to stderr and the process exits with code 1,
which MCP clients show in their server log.

## License

MIT for this package. The bundled open-source packages and the Nunito font keep their own
licenses, listed in `dist/third-party-notices.txt`.
