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

    // Option 1: Management bearer token (API key or JWT)
    // For key lifecycle endpoints (list/rotate/revoke), use a JWT access token.
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

- `TETHER_API_KEY` (management bearer token; API key or JWT)
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

// Create an agent with a verified domain
domains, err := client.ListDomains(ctx)
if len(domains) > 0 && domains[0].Verified {
    agent, err := client.CreateAgent(ctx, "my-bot", "My bot", domains[0].ID)
    // Verification results will show the domain instead of email
}

// List all agents
agents, err := client.ListAgents(ctx)

// Delete an agent
deleted, err := client.DeleteAgent(ctx, agent.ID)
fmt.Println(deleted)

// Key lifecycle endpoints require JWT auth (not API key).
// You can pass a JWT access token in Options.ApiKey for these calls.
// List key lifecycle entries
keys, err := client.ListAgentKeys(ctx, agent.ID)
fmt.Println(len(keys))

// Rotate key (requires step-up)
rotated, err := client.RotateAgentKey(ctx, agent.ID, tether.RotateAgentKeyRequest{
    PublicKey: "BASE64_SPKI_PUBLIC_KEY",
    GracePeriodHours: 24,
    StepUpCode: "123456", // or Challenge + Proof
})

// Revoke a key
_, err = client.RevokeAgentKey(ctx, agent.ID, rotated.NewKeyID, tether.RevokeAgentKeyRequest{
    Reason: "compromised",
    StepUpCode: "654321",
})
```

## Domain Management

List domains registered to your account. Domains are claimed and verified via the [web dashboard](https://tether.name/dashboard) or [API](../api/domains.md), then referenced by ID when creating agents.

```go
client, err := tether.NewClient(tether.Options{
    ApiKey: "sk-tether-name-...",
})

// List all domains
domains, err := client.ListDomains(ctx)
for _, domain := range domains {
    status := "pending"
    if domain.Verified {
        status = "verified"
    }
    fmt.Printf("%s — %s\n", domain.Domain, status)
}
```

## API

### `tether.NewClient(opts Options) (*TetherClient, error)`

Creates a new client. Returns an error if neither `ApiKey` nor `AgentID` is provided.

### `client.CreateAgent(ctx, agentName, description, domainID...) (*Agent, error)`

Create a new agent. Optionally pass a verified domain ID as the last argument. Requires API key auth.

### `client.ListAgents(ctx) ([]Agent, error)`

List all agents. Requires API key auth.

### `client.DeleteAgent(ctx, agentID) (bool, error)`

Delete an agent. Requires API key auth.

### `client.ListDomains(ctx) ([]Domain, error)`

List all registered domains for the authenticated account. Requires API key auth.

### `client.ListAgentKeys(ctx, agentID) ([]AgentKey, error)`

List key lifecycle entries (`active`, `grace`, `revoked`) for an agent. Requires JWT access token auth.

### `client.RotateAgentKey(ctx, agentID, req) (*RotateAgentKeyResponse, error)`

Rotate an agent key. Requires JWT access token auth plus step-up verification via either `StepUpCode` or `Challenge` + `Proof`.

### `client.RevokeAgentKey(ctx, agentID, keyID, req) (*RevokeAgentKeyResponse, error)`

Revoke an agent key. Requires JWT access token auth plus step-up verification via either `StepUpCode` or `Challenge` + `Proof`.

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
    Domain          string     // Verified domain (if assigned to this agent)
    RegisteredSince *EpochTime
    Error           string
    Challenge       string
}

type Agent struct {
    ID                string
    AgentName         string
    Description       string
    DomainID          string
    Domain            string
    CreatedAt         int64
    RegistrationToken string
    LastVerifiedAt    int64
}

type Domain struct {
    ID            string
    Domain        string
    Verified      bool
    VerifiedAt    int64
    LastCheckedAt int64
    CreatedAt     int64
}

type AgentKey struct {
    ID            string
    Status        string // active | grace | revoked
    CreatedAt     int64
    ActivatedAt   int64
    GraceUntil    int64
    RevokedAt     int64
    RevokedReason string
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
