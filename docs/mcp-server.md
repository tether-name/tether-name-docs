# MCP Server

[![npm](https://img.shields.io/npm/v/tether-name-mcp-server)](https://www.npmjs.com/package/tether-name-mcp-server)

The Tether MCP server lets any MCP-compatible AI agent verify its identity without writing integration code. Just add it to your MCP config.

## Quick Start

```bash
npx tether-name-mcp-server
```

## Setup

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "tether": {
      "command": "npx",
      "args": ["-y", "tether-name-mcp-server"],
      "env": {
        "TETHER_AGENT_ID": "your-agent-id",
        "TETHER_PRIVATE_KEY_PATH": "/path/to/private-key.der",
        "TETHER_API_KEY": "sk-tether-name-..."
      }
    }
  }
}
```

!!! tip
    `TETHER_API_KEY` is only needed for management tools (`create_agent`, `list_agents`, `delete_agent`, `list_domains`, `list_agent_keys`, `rotate_agent_key`, `revoke_agent_key`).

    API keys work for these tools. For key lifecycle mutations (`rotate_agent_key`, `revoke_agent_key`), prefer `challenge` + `proof` step-up in automation.

    `request_challenge` does **not** require signing env vars. `verify_identity`, `sign_challenge`, and `submit_proof` require both `TETHER_AGENT_ID` and `TETHER_PRIVATE_KEY_PATH`.

### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "tether": {
      "command": "npx",
      "args": ["-y", "tether-name-mcp-server"],
      "env": {
        "TETHER_AGENT_ID": "your-agent-id",
        "TETHER_PRIVATE_KEY_PATH": "/path/to/private-key.der"
      }
    }
  }
}
```

### VS Code

Add to `.vscode/mcp.json`:

```json
{
  "mcp": {
    "servers": {
      "tether": {
        "command": "npx",
        "args": ["-y", "tether-name-mcp-server"],
        "env": {
          "TETHER_AGENT_ID": "your-agent-id",
          "TETHER_PRIVATE_KEY_PATH": "/path/to/private-key.der"
        }
      }
    }
  }
}
```

## Tools

### Verification

| Tool | Description |
|---|---|
| `verify_identity` | Complete verification flow — requests a challenge, signs it, submits proof |
| `request_challenge` | Request a new challenge from the Tether API |
| `sign_challenge` | Sign a challenge with the configured RSA private key |
| `submit_proof` | Submit a signed proof for verification |
| `get_agent_info` | Show configured agent ID and key path |

### Agent Management

These tools require `TETHER_API_KEY` to be set (as a management bearer token: API key or JWT).

| Tool | Description |
|---|---|
| `create_agent` | Create a new agent (with optional domain assignment) |
| `list_agents` | List all agents for the authenticated account |
| `delete_agent` | Delete an agent by ID |
| `list_domains` | List all registered domains |
| `list_agent_keys` | List key lifecycle entries (`active`, `grace`, `revoked`) for an agent |
| `rotate_agent_key` | Rotate an agent key (requires step-up: `stepUpCode` or `challenge` + `proof`) |
| `revoke_agent_key` | Revoke an agent key (requires step-up: `stepUpCode` or `challenge` + `proof`) |

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `TETHER_AGENT_ID` | For sign/submit/verify tools | Your Tether agent ID |
| `TETHER_PRIVATE_KEY_PATH` | For sign/submit/verify tools | Path to RSA private key (DER or PEM) |
| `TETHER_API_KEY` | For management | Management bearer token (API key or JWT) |
| `TETHER_API_URL` | Optional | Override API URL (default: `https://api.tether.name`) |

## Security

The private key stays on your machine. The MCP server reads it from a local file — it's never transmitted. The server runs as a local subprocess via STDIO.

## Links

- [npm](https://www.npmjs.com/package/tether-name-mcp-server)
- [GitHub](https://github.com/tether-name/tether-name-mcp-server)
