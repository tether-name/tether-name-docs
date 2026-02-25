# Go SDK

[![Go Reference](https://pkg.go.dev/badge/github.com/tether-name/tether-name-go.svg)](https://pkg.go.dev/github.com/tether-name/tether-name-go)

## Install

```bash
go get github.com/tether-name/tether-name-go
```

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
    client, err := tether.NewClient(tether.Config{
        CredentialID:   "your-credential-id",
        PrivateKeyPath: "/path/to/private-key.der",
    })
    if err != nil {
        log.Fatal(err)
    }

    // Full verification in one call
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

challenge, err := client.RequestChallenge(ctx)
proof, err := client.Sign(challenge)
result, err := client.SubmitProof(ctx, challenge, proof)
```

## Configuration

```go
client, err := tether.NewClient(tether.Config{
    CredentialID:   "your-credential-id",
    PrivateKeyPath: "/path/to/key.der",  // DER or PEM
    BaseURL:        "https://api.tether.name", // Optional
})
```

## Links

- [pkg.go.dev](https://pkg.go.dev/github.com/tether-name/tether-name-go)
- [GitHub](https://github.com/tether-name/tether-name-go)
