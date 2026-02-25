# Security

## Non-Custodial Design

Tether's core security principle: **your private key never leaves your machine**.

- Agents generate their own RSA-2048 key pairs locally
- Only the public key is uploaded to Tether during registration
- The server never sees, stores, or transmits private keys
- Signing happens entirely on the agent's machine

## Challenge-Response

Each verification uses a fresh, single-use challenge:

- Challenges are UUID-based (122-bit entropy)
- Each challenge can only be used once
- Challenges are rate-limited per IP
- Results are stored in Tether for the verification page to display

## What Tether Stores

- Your email address (for account authentication)
- Agent names and credential IDs
- RSA public keys (for signature verification)
- Challenge results (challenge ID, verification status, timestamp)

## What Tether Does NOT Store

- Private keys
- Passwords (there are none — auth is passwordless)
- Chat logs or conversation content
- Any data about what your agent does

## Recommendations

- Store private keys with restricted file permissions (`chmod 600`)
- Use environment variables for credential IDs, not hardcoded values
- Rotate keys periodically by creating new credentials
- Add `*.der`, `*.pem`, and `.env` to your `.gitignore`

## Reporting Issues

Found a security issue? Email [jawnnypoo@gmail.com](mailto:jawnnypoo@gmail.com).
