# Quick Start

Get your agent verifying its identity in under 5 minutes.

## Prerequisites

- A Tether credential ([register here](register.md))
- Your agent's private key file

## Verify Identity

=== "Node.js"

    ```bash
    npm install tether-name
    ```

    ```typescript
    import { TetherClient } from 'tether-name';

    const client = new TetherClient({
      credentialId: process.env.TETHER_CREDENTIAL_ID,
      privateKeyPath: process.env.TETHER_PRIVATE_KEY_PATH
    });

    const result = await client.verify();
    console.log(result.verified);   // true
    console.log(result.agentName);  // "My Agent"
    console.log(result.verifyUrl);  // "https://tether.name/check?challenge=..."
    ```

=== "Python"

    ```bash
    pip install tether-name
    ```

    ```python
    from tether_name import TetherClient

    client = TetherClient(
        credential_id="your-credential-id",
        private_key_path="/path/to/private-key.der"
    )

    result = client.verify()
    print(result.verified)    # True
    print(result.agent_name)  # "My Agent"
    print(result.verify_url)  # "https://tether.name/check?challenge=..."
    ```

=== "Go"

    ```bash
    go get github.com/tether-name/tether-name-go
    ```

    ```go
    package main

    import (
        "fmt"
        tether "github.com/tether-name/tether-name-go"
    )

    func main() {
        client, _ := tether.NewClient(tether.Options{
            CredentialID:   "your-credential-id",
            PrivateKeyPath: "/path/to/private-key.der",
        })

        result, _ := client.Verify(context.Background())
        fmt.Println(result.Verified)   // true
        fmt.Println(result.AgentName)  // "My Agent"
        fmt.Println(result.VerifyURL)  // "https://tether.name/check?challenge=..."
    }
    ```

=== "CLI"

    ```bash
    npm install -g tether-name-cli
    ```

    ```bash
    # Interactive setup
    tether init

    # Verify identity
    tether verify
    ```

=== "MCP"

    No code needed — just add to your MCP config:

    ```json
    {
      "mcpServers": {
        "tether": {
          "command": "npx",
          "args": ["-y", "tether-name-mcp"],
          "env": {
            "TETHER_CREDENTIAL_ID": "your-credential-id",
            "TETHER_PRIVATE_KEY_PATH": "/path/to/private-key.der"
          }
        }
      }
    }
    ```

    Then your agent can call the `verify_identity` tool.

## Responding to a Challenge

When a user sends your agent a challenge code, sign and submit it:

=== "Node.js"

    ```typescript
    const challenge = "the-challenge-code-from-user";
    const proof = client.sign(challenge);
    const result = await client.submitProof(challenge, proof);
    ```

=== "Python"

    ```python
    challenge = "the-challenge-code-from-user"
    proof = client.sign(challenge)
    result = client.submit_proof(challenge, proof)
    ```

=== "Go"

    ```go
    challenge := "the-challenge-code-from-user"
    proof, _ := client.Sign(challenge)
    result, _ := client.SubmitProof(context.Background(), challenge, proof)
    ```

=== "CLI"

    ```bash
    tether sign "the-challenge-code-from-user"
    tether check "the-challenge-code-from-user"
    ```

## Environment Variables

All SDKs support configuration via environment variables:

| Variable | Used by | Description |
|---|---|---|
| `TETHER_CREDENTIAL_ID` | SDKs, CLI, MCP | Your credential ID |
| `TETHER_PRIVATE_KEY_PATH` | SDKs, CLI, MCP | Path to your RSA private key |
