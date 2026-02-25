# Tether.name

**AI agent identity verification — so you know who you're talking to.**

Tether lets AI agents prove their identity through cryptographic challenge-response verification. No passwords, no custodial keys — agents hold their own RSA private keys and sign challenges to prove they are who they say they are.

## For Users

Want to verify an agent? Head to [tether.name](https://tether.name) — no account needed. Click "Verify an Agent," send the challenge to the agent, and watch the result appear in real time.

## For Developers

Building an agent that needs to prove its identity? You're in the right place.

<div class="grid cards" markdown>

-   :material-lightning-bolt: **Quick Start**

    ---

    Get your agent verified in under 5 minutes.

    [:octicons-arrow-right-24: Getting Started](getting-started/quickstart.md)

-   :material-package-variant: **SDKs**

    ---

    Official libraries for Node.js, Python, and Go.

    [:octicons-arrow-right-24: SDKs](sdks/node.md)

-   :material-robot: **MCP Server**

    ---

    Drop-in identity verification for any MCP-compatible agent.

    [:octicons-arrow-right-24: MCP Server](mcp-server.md)

-   :material-api: **API Reference**

    ---

    Full REST API documentation.

    [:octicons-arrow-right-24: API Reference](api/challenges.md)

</div>

## How It Works

```
┌─────────┐         ┌──────────────┐         ┌─────────┐
│  Agent   │───1────▶│ api.tether   │◀───2────│  User   │
│          │         │   .name      │         │         │
│  Signs   │───3────▶│  Verifies    │───4────▶│  Sees   │
│ challenge│         │   proof      │         │ result  │
└─────────┘         └──────────────┘         └─────────┘
```

1. User requests a challenge from Tether
2. User sends the challenge to the agent
3. Agent signs the challenge with its private key and submits proof
4. Tether verifies the signature and shows the result

The private key never leaves the agent's machine.

## Install

=== "Node.js"

    ```bash
    npm install tether-name
    ```

=== "Python"

    ```bash
    pip install tether-name
    ```

=== "Go"

    ```bash
    go get github.com/tether-name/tether-name-go
    ```

=== "MCP"

    ```json
    {
      "mcpServers": {
        "tether": {
          "command": "npx",
          "args": ["-y", "tether-name-mcp"]
        }
      }
    }
    ```
