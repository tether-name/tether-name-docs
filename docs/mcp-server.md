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
        "TETHER_CREDENTIAL_ID": "your-credential-id",
        "TETHER_PRIVATE_KEY_PATH": "/path/to/private-key.der"
      }
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "tether": {
      "command": "npx",
      "args": ["-y", "tether-name-mcp-server"],
      "env": {
        "TETHER_CREDENTIAL_ID": "your-credential-id",
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
          "TETHER_CREDENTIAL_ID": "your-credential-id",
          "TETHER_PRIVATE_KEY_PATH": "/path/to/private-key.der"
        }
      }
    }
  }
}
```

## Tools

| Tool | Description |
|---|---|
| `verify_identity` | Complete verification flow — requests a challenge, signs it, submits proof |
| `request_challenge` | Request a new challenge from the Tether API |
| `sign_challenge` | Sign a challenge with the configured RSA private key |
| `submit_proof` | Submit a signed proof for verification |
| `get_credential_info` | Show configured credential ID and key path |

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `TETHER_CREDENTIAL_ID` | ✅ | Your Tether credential ID |
| `TETHER_PRIVATE_KEY_PATH` | ✅ | Path to RSA private key (DER or PEM) |

## Security

The private key stays on your machine. The MCP server reads it from a local file — it's never transmitted. The server runs as a local subprocess via STDIO.

## Links

- [npm](https://www.npmjs.com/package/tether-name-mcp-server)
- [GitHub](https://github.com/tether-name/tether-name-mcp-server)
