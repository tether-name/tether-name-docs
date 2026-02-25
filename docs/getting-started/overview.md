# Overview

Tether.name provides identity verification for AI agents using RSA cryptographic challenge-response.

## Core Concepts

### Credentials

When you register an agent on [tether.name](https://tether.name), you receive a **credential** consisting of:

- **Credential ID** — A unique identifier for your agent
- **Registration Token** — A one-time token used to register your agent's public key

### Key Pairs

Each agent generates its own RSA-2048 key pair:

- **Private key** — Stays on your agent's machine. Used to sign challenges. Never shared.
- **Public key** — Uploaded to Tether during registration. Used to verify signatures.

### Challenges

A challenge is a UUID-based code (122-bit entropy) that Tether generates for each verification attempt. The agent signs the challenge with its private key, and Tether verifies the signature against the registered public key.

## Security Model

- **Non-custodial** — Tether never sees your private key. Your agent generates the key pair and only uploads the public key.
- **No replay attacks** — Each challenge is single-use and rate-limited.
- **Traceable** — Every agent links back to a verified email address.

## Flow

1. **Register** — Create an account on tether.name, name your agent, and receive credentials
2. **Generate keys** — Your agent generates an RSA-2048 key pair and registers the public key with Tether
3. **Verify** — When challenged, your agent signs the challenge with its private key and submits the proof

## Next Steps

- [Register your agent](register.md)
- [Quick start guide](quickstart.md)
