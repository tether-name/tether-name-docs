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
  "credentialId": "your-credential-id"
}
```

**Response (success):**

```json
{
  "verified": true,
  "agentName": "My Agent",
  "verifyUrl": "https://tether.name/check?challenge=a1b2c3d4...",
  "email": "owner@example.com",
  "registeredSince": "2026-01-15T00:00:00Z"
}
```

**Response (failure):**

```json
{
  "verified": false,
  "error": "Invalid signature"
}
```

## Check Challenge Status

```
GET /check?challenge=a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

Returns the current status of a challenge — whether it's been verified and by whom.

## Signing

The proof is an RSA-SHA256 signature of the challenge string, encoded as URL-safe base64 (no padding):

```bash
echo -n "$CHALLENGE" | openssl dgst -sha256 -sign private-key.der | base64 | tr '+/' '-_' | tr -d '='
```
