# Go SDK

[![Go Reference](https://pkg.go.dev/badge/github.com/tether-name/tether-name-go.svg)](https://pkg.go.dev/github.com/tether-name/tether-name-go)

## Install

```bash
go get github.com/tether-name/tether-name-go
```

Requires Go 1.22+. Zero external dependencies — uses only the Go standard library.

## Usage

```go
package main

import (
    "context"
    "fmt"
    "log"

    tether "github.com/tether-name/tether-name-go"
)

func main() {
    client, err := tether.NewClient(tether.Options{
        AgentID:   "your-agent-id",
        PrivateKeyPath: "/path/to/private-key.der",
    })
    if err != nil {
        log.Fatal(err)
    }

    result, err := client.Verify(context.Background())
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(result.Verified)
    fmt.Println(result.AgentName)
}
```

## Step-by-Step Verification

```go
ctx := context.Background()

// 1. Request a challenge
challenge, err := client.RequestChallenge(ctx)

// 2. Sign the challenge
proof, err := client.Sign(challenge)

// 3. Submit proof
result, err := client.SubmitProof(ctx, challenge, proof)
```

## Configuration

```go
client, err := tether.NewClient(tether.Options{
    // Authentication — choose one:

    // Option 1: API key (for agent management and agent operations)
    ApiKey: "sk-tether-name-...",

    // Option 2: Agent + private key (for verification and signing)
    AgentID: "your-agent-id",
    PrivateKeyPath: "/path/to/key.der",  // File path (DER or PEM)
    PrivateKeyPEM:  pemBytes,             // PEM as []byte
    PrivateKeyDER:  derBytes,             // DER as []byte
})
```

When `ApiKey` is set, `AgentID` and private key options become optional. A private key is still required for `Verify()` and `Sign()`.

Environment variables:

- `TETHER_API_KEY`
- `TETHER_AGENT_ID`
- `TETHER_PRIVATE_KEY_PATH`

## Agent Management

One line to start managing agents programmatically:

```go
client, err := tether.NewClient(tether.Options{
    ApiKey: "sk-tether-name-...",
})

// Create an agent
agent, err := client.CreateAgent(ctx, "my-bot", "My bot description")
fmt.Println(agent.ID)

// List all agents
agents, err := client.ListAgents(ctx)

// Delete an agent
deleted, err := client.DeleteAgent(ctx, agent.ID)
fmt.Println(deleted)
```

## API

### `tether.NewClient(opts Options) (*TetherClient, error)`

Creates a new client. Returns an error if neither `ApiKey` nor `AgentID` is provided.

### `client.CreateAgent(ctx, agentName, description) (*Agent, error)`

Create a new agent. Requires API key auth.

### `client.ListAgents(ctx) ([]Agent, error)`

List all agents. Requires API key auth.

### `client.DeleteAgent(ctx, agentID) (bool, error)`

Delete an agent. Requires API key auth.

### `client.Verify(ctx) (*VerificationResult, error)`

Complete verification flow. Requests a challenge, signs it, and submits proof.

### `client.RequestChallenge(ctx) (string, error)`

Request a new challenge from the Tether API.

### `client.Sign(challenge) (string, error)`

Sign a challenge string with the configured private key.

Returns: URL-safe base64 signature (no padding).

### `client.SubmitProof(ctx, challenge, proof) (*VerificationResult, error)`

Submit a signed proof for verification.

## Types

```go
type VerificationResult struct {
    Verified        bool
    AgentName       string
    VerifyURL       string
    Email           string
    RegisteredSince *EpochTime
    Error           string
    Challenge       string
}
```

## Error Handling

```go
import "errors"

result, err := client.Verify(ctx)
if err != nil {
    var apiErr *tether.APIError
    var verifyErr *tether.VerificationError
    var keyErr *tether.KeyLoadError

    switch {
    case errors.As(err, &apiErr):
        fmt.Printf("API error (status %d): %s\n", apiErr.StatusCode, apiErr.Message)
    case errors.As(err, &verifyErr):
        fmt.Printf("Verification failed: %s\n", verifyErr.Message)
    case errors.As(err, &keyErr):
        fmt.Printf("Key error: %s\n", keyErr.Message)
    }
}
```

Sentinel errors for `errors.Is()`:

- `tether.ErrAPI` — API communication error
- `tether.ErrVerification` — Verification failure
- `tether.ErrKeyLoad` — Private key loading error

## Links

- [pkg.go.dev](https://pkg.go.dev/github.com/tether-name/tether-name-go)
- [GitHub](https://github.com/tether-name/tether-name-go)
