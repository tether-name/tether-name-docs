# Challenges

Challenges are the core of Tether's verification flow. They are single-use, rate-limited, and expire after a short window.

## Request a Challenge

No authentication required.

```
POST /challenge
Content-Type: application/json
```

**Response:**

```json
{
  "code": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

## Verify a Challenge

Submit a signed proof to verify an agent's identity. No authentication required.

```
POST /challenge/verify
Content-Type: application/json

{
  "challenge": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "proof": "url-safe-base64-signature",
  "agentId": "your-agent-id"
}
```

**Response (success):**

```json
{
  "verified": true,
  "agentName": "My Agent",
  "verifyUrl": "https://tether.name/check?challenge=a1b2c3d4...",
  "domain": "example.com",
  "registeredSince": 1736899200000
}
```

!!! note
    `registeredSince` is a Unix timestamp in milliseconds (epoch ms).

!!! info
    The response includes `domain` if the agent's owner has a [verified domain](domains.md), or `email` otherwise. Only one will be present.

**Response (failure):**

```json
{
  "verified": false,
  "error": "Invalid signature"
}
```

## Check Challenge Status

```
GET /challenge/{code}
```

Returns the current status of a challenge — whether it's been verified and by whom.

**Response (pending):**

```json
{
  "challenge": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "pending",
  "createdAt": 1736899200000,
  "poll": {
    "intervalMs": 3000,
    "maxAttempts": 60
  }
}
```

**Response (verified — with domain):**

```json
{
  "challenge": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "verified",
  "createdAt": 1736899200000,
  "agentName": "My Agent",
  "domain": "example.com",
  "registeredSince": 1736899200000,
  "verifiedAt": 1736899260000
}
```

**Response (verified — with email):**

```json
{
  "challenge": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "verified",
  "createdAt": 1736899200000,
  "agentName": "My Agent",
  "email": "owner@example.com",
  "registeredSince": 1736899200000,
  "verifiedAt": 1736899260000
}
```

!!! tip
    The frontend check page at `https://tether.name/check?challenge=...` polls this endpoint to show verification status to users.

## Signing

The proof is an RSA-SHA256 signature of the challenge string, encoded as URL-safe base64 (no padding):

```bash
echo -n "$CHALLENGE" | openssl dgst -sha256 -sign private-key.der | base64 | tr '+/' '-_' | tr -d '='
```
