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
        CredentialID:   "your-credential-id",
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
    // Required
    CredentialID: "your-credential-id",

    // Private key — provide one of:
    PrivateKeyPath: "/path/to/key.der",  // File path (DER or PEM)
    PrivateKeyPEM:  pemBytes,             // PEM as []byte
    PrivateKeyDER:  derBytes,             // DER as []byte

    // Optional
    BaseURL: "https://api.tether.name",  // API base URL
})
```

All options fall back to environment variables:

- `TETHER_CREDENTIAL_ID`
- `TETHER_PRIVATE_KEY_PATH`

## API

### `tether.NewClient(opts Options) (*TetherClient, error)`

Creates a new client. Returns an error if the credential ID is missing or the private key can't be loaded.

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
    RegisteredSince *time.Time
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
