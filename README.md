# MultiVoice MCP

Remote [Model Context Protocol](https://modelcontextprotocol.io/) server for **AI video dubbing**, **subtitles**, **transcription**, and **multi-voice audiobooks**.

Exact price is shown and must be confirmed by a human **before** any paid job starts.

- **Docs:** https://multivoice.ai/for-agents
- **Endpoint:** `https://db.multivoice.ai/mcp`
- **Glama:** [Video Dubbing, Subtitles, Transcription & Audiobooks — MultiVoice](https://glama.ai/mcp/connectors/ai.multivoice/video-dubbing-subtitles-transcription-audiobooks)
- **OpenAPI / Swagger:** https://db.multivoice.ai

This repository contains documentation only. The MCP server is hosted; there is no installable local package here.

## Endpoint

| | |
| --- | --- |
| URL | `https://db.multivoice.ai/mcp` |
| Transport | Streamable HTTP (POST). Responses may be JSON or SSE-framed. |
| Auth | OAuth 2.1 Authorization Code + PKCE S256 (public client) |
| Note | `GET /mcp` returns **405** with `Allow: POST` — expected |

Never put API keys or JWTs in config files. Complete browser OAuth once (login + Allow access).

## Cursor

### One-click install

[Add MultiVoice in Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=multivoice&config=eyJ1cmwiOiJodHRwczovL2RiLm11bHRpdm9pY2UuYWkvbWNwIn0=)

### Project or global `mcp.json`

Create or edit `.cursor/mcp.json` (project) or your Cursor MCP settings:

```json
{
  "mcpServers": {
    "multivoice": {
      "url": "https://db.multivoice.ai/mcp"
    }
  }
}
```

1. Save the file and reload MCP / restart Cursor if needed.
2. Complete human OAuth in the browser when prompted.
3. Smoke test: call `multivoice.account.get`.

## Claude Desktop

Claude Desktop does **not** load remote MCP URLs from `claude_desktop_config.json`. Use **Connectors**:

1. Claude Desktop → **Settings** → **Connectors** → **Add custom connector**.
2. URL: `https://db.multivoice.ai/mcp`
3. Complete browser OAuth as a human.
4. Smoke test: `multivoice.account.get`.

### Shared `mcpServers` JSON (Cursor / clients that support remote URL config)

Some clients accept the same shape as Cursor. Example (do **not** expect Claude Desktop to honor this file for remote servers):

```json
{
  "mcpServers": {
    "multivoice": {
      "url": "https://db.multivoice.ai/mcp"
    }
  }
}
```

## Claude Code (CLI)

```bash
claude mcp add --transport http multivoice https://db.multivoice.ai/mcp
```

Then complete human OAuth when the CLI opens the browser.

## Codex

```bash
codex mcp add multivoice --url https://db.multivoice.ai/mcp
codex mcp login multivoice
```

## Security: `confirmation_handle`

Paid creates never run on agent text alone. The server enforces a quote + human confirmation gate:

```text
get_create_options
       ↓
prepare_create  →  commitment_amount, quote_id, confirmation_handle
       ↓
Human confirms the shown amount in the MCP client
       ↓
execute_create(quote_id, confirmation_handle, operation_id)
```

### Rules

1. **`orders.prepare_create`** returns a short-lived quote (`commitment_amount`, `quote_id`, `confirmation_handle`).
2. The agent must **show the amount to the human** and wait for confirmation of that price in the MCP client.
3. Only then call **`orders.execute_create`** with:
   - `quote_id`
   - `confirmation_handle` (from prepare)
   - `operation_id` (new UUID; **reuse the same id on retry** — does not create a second charge)
4. **`execute_create` without a valid `confirmation_handle` is rejected** (`validation_error`). Agents cannot bypass this with prompt text.
5. Browser OAuth is **one-time** (login + Allow once). Do **not** open a browser again for each order — only confirm the shown amount in the MCP client.
6. Never invent prices. Only use `commitment_amount` from `prepare_create`.

### Minimal paid flow (illustration)

```text
// 1) Options / languages
multivoice.orders.get_create_options { "order_type_id": 4 }

// 2) Quote (example: subtitles)
multivoice.orders.prepare_create { ... }

// 3) Show commitment_amount → wait for human confirm in MCP client

// 4) Charge only after confirm
multivoice.orders.execute_create {
  "quote_id": "...",
  "confirmation_handle": "...",
  "operation_id": "<new UUID; reuse on retry>"
}
```

Order types via MCP: **1** audiobook, **4** subtitles, **5** transcription, **7** dubbing. Full tool list and schemas: https://multivoice.ai/for-agents

## Links

| Resource | URL |
| --- | --- |
| Agent docs | https://multivoice.ai/for-agents |
| MCP endpoint | https://db.multivoice.ai/mcp |
| Glama connector | https://glama.ai/mcp/connectors/ai.multivoice/video-dubbing-subtitles-transcription-audiobooks |
| OpenAPI | https://db.multivoice.ai |
| Website | https://multivoice.ai |

## License

Documentation in this repository is provided for integrating with the hosted MultiVoice MCP service. The server implementation remains proprietary.
