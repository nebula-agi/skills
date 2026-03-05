# MCP Integration Setup

Connect AI assistants to Nebula via the web-hosted MCP server at `https://mcp.trynebula.ai/mcp`.

## Prerequisites

- API key from [trynebula.ai](https://trynebula.ai) -> Settings -> API Keys
- A collection ID (find in your collection's settings)

Quickest setup: use the Connect dialog on your collection page at trynebula.ai to auto-generate config.

## Client Configurations

### Cursor

File: `.cursor/mcp.json`

```json
{
  "mcpServers": {
    "nebula-memory": {
      "url": "https://mcp.trynebula.ai/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY",
        "X-Collection-ID": "YOUR_COLLECTION_ID"
      }
    }
  }
}
```

### VS Code

File: `.vscode/mcp.json`

```json
{
  "servers": {
    "nebula-memory": {
      "type": "http",
      "url": "https://mcp.trynebula.ai/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY",
        "X-Collection-ID": "YOUR_COLLECTION_ID"
      }
    }
  }
}
```

Note: VS Code uses `"servers"` as the top-level key, not `"mcpServers"`.

### Windsurf

File: `~/.codeium/windsurf/mcp_config.json`

```json
{
  "mcpServers": {
    "nebula-memory": {
      "serverUrl": "https://mcp.trynebula.ai/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY",
        "X-Collection-ID": "YOUR_COLLECTION_ID"
      }
    }
  }
}
```

Note: Windsurf uses `"serverUrl"` instead of `"url"`.

### Claude Desktop

File: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows)

Uses `mcp-remote` bridge for stdio transport:

```json
{
  "mcpServers": {
    "nebula-memory": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote",
        "https://mcp.trynebula.ai/mcp",
        "--header", "Authorization: Bearer ${NEBULA_API_KEY}",
        "--header", "X-Collection-ID: YOUR_COLLECTION_ID"
      ],
      "env": {
        "NEBULA_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

### Claude Code

CLI install:

```bash
claude mcp add --transport http --scope user nebula-memory https://mcp.trynebula.ai/mcp \
  --header "Authorization: Bearer YOUR_API_KEY" \
  --header "X-Collection-ID: YOUR_COLLECTION_ID"
```

Or manually in `~/.claude.json`:

```json
{
  "mcpServers": {
    "nebula-memory": {
      "transport": "http",
      "url": "https://mcp.trynebula.ai/mcp",
      "httpHeaders": {
        "Authorization": "Bearer YOUR_API_KEY",
        "X-Collection-ID": "YOUR_COLLECTION_ID"
      }
    }
  }
}
```

## Available MCP Tools

### `add_memory`
Store documents or conversation messages.

- `content` (required): Text to store
- `role` (optional): `user` or `assistant` for conversations
- `metadata` (optional): Custom metadata object

### `search_memories`
Semantic search across stored memories.

- `query` (required): Search text
- `effort` (optional): `auto`/`low`/`medium`/`high`

## Local Installation (Optional)

```bash
npm install -g @nebula-ai/mcp-server
```

Use stdio transport with env vars `NEBULA_API_KEY` and `NEBULA_API_URL`.

## Troubleshooting

- **Tools not appearing**: Restart the AI client after config changes. Verify API key and collection ID.
- **Connection errors**: Ensure firewall allows `mcp.trynebula.ai`. Check client logs.
- **Auth issues**: Include `Bearer ` prefix in Authorization header. Verify collection exists.
