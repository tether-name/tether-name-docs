# Agents

Agents represent a registered AI agent. Each agent has a unique ID and an associated RSA public key.

## Create an Agent

Requires authentication (JWT or API key).

```
POST /agents/issue
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "agentName": "My Agent",
  "description": "Optional description for this agent",
  "domainId": "optional-verified-domain-id"
}
```

**Response (201 Created):**

```json
{
  "id": "rgUOzbqar8z0Ag9RZH5I",
  "agentName": "My Agent",
  "description": "Optional description for this agent",
  "domainId": "optional-verified-domain-id",
  "createdAt": 1736899200000,
  "registrationToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

!!! warning
    The `registrationToken` is shown only once. Save it — you'll need it to register your public key. It expires after 5 minutes.

!!! info
    `domainId` is optional. If provided, it must reference one of your **verified** domains. This agent will then display **that exact domain** on verification results. If omitted, verification falls back to the owner's email (no automatic domain fallback).

## Register a Public Key

No authentication required — uses the registration token instead.

The `publicKey` must be a **base64-encoded DER** (SubjectPublicKeyInfo / X.509) RSA public key, minimum 2048 bits.

```
POST /agents/{agentId}/register-key
Content-Type: application/json

{
  "registrationToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "publicKey": "MIIBIjANBgkqhki..."
}
```

**Response:**

```json
{
  "id": "rgUOzbqar8z0Ag9RZH5I",
  "agentName": "My Agent",
  "registered": true
}
```

!!! info
    `register-key` is for first key registration only. After initial setup, use key lifecycle endpoints (`/agents/{agentId}/keys/rotate`, `/agents/{agentId}/keys/{keyId}/revoke`).

## Check Agent Status

Requires authentication (JWT or API key).

```
GET /agents/{agentId}/status
Authorization: Bearer eyJ...
```

**Response:**

```json
{
  "id": "rgUOzbqar8z0Ag9RZH5I",
  "agentName": "My Agent",
  "registered": true
}
```

## List Agent Keys

Requires JWT authentication.

```
GET /agents/{agentId}/keys
Authorization: Bearer eyJ...
```

**Response:**

```json
[
  {
    "id": "key_abc",
    "status": "active",
    "createdAt": 1736899200000,
    "activatedAt": 1736899200000,
    "graceUntil": 0,
    "revokedAt": 0,
    "revokedReason": ""
  }
]
```

## Rotate Agent Key

Requires JWT authentication **plus step-up verification**.

Provide one of:
- `stepUpCode` (email code from auth flow), or
- `challenge` + `proof` (signed by an active/grace key)

```
POST /agents/{agentId}/keys/rotate
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "publicKey": "MIIBIjANBgkqhki...",
  "gracePeriodHours": 24,
  "reason": "routine_rotation",
  "stepUpCode": "123456"
}
```

**Response:**

```json
{
  "agentId": "rgUOzbqar8z0Ag9RZH5I",
  "previousKeyId": "key_old",
  "newKeyId": "key_new",
  "graceUntil": 1736985600000,
  "message": "Key rotated successfully"
}
```

## Revoke Agent Key

Requires JWT authentication **plus step-up verification**.

Provide one of:
- `stepUpCode`, or
- `challenge` + `proof`

```
POST /agents/{agentId}/keys/{keyId}/revoke
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "reason": "compromised",
  "stepUpCode": "123456"
}
```

**Response:**

```json
{
  "agentId": "rgUOzbqar8z0Ag9RZH5I",
  "keyId": "key_new",
  "revoked": true,
  "promotedKeyId": "key_old",
  "message": "Key revoked. A grace key was promoted to active"
}
```

## List Agents

Requires authentication (JWT or API key).

```
GET /agents
Authorization: Bearer eyJ...
```

**Response:**

```json
[
  {
    "id": "rgUOzbqar8z0Ag9RZH5I",
    "agentName": "My Agent",
    "description": "Optional description",
    "domainId": "optional-verified-domain-id",
    "domain": "example.com",
    "createdAt": 1736899200000,
    "lastVerifiedAt": 1736985600000
  }
]
```

## Delete an Agent

Requires authentication (JWT or API key).

```
DELETE /agents/{agentId}
Authorization: Bearer eyJ...
```

**Response:**

```json
{
  "message": "Agent deleted"
}
```

## Limits

| Limit | Value |
|-------|-------|
| Agents per account | 10 |
| Agent name length | 100 characters |
| Description length | 500 characters |
| Rate limit (CRUD) | 20 requests/minute |
| Rate limit (status) | 60 requests/minute |
| Rate limit (register-key) | 5 requests/10 minutes |
| Rate limit (keys rotate) | 3 requests/10 minutes |
| Rate limit (keys revoke) | 3 requests/10 minutes |

!!! info
    If you need more agents, please contact us at [support@tether.name](mailto:support@tether.name).

## Key Generation

Generate an RSA-2048 key pair for your agent:

```bash
# Generate private key (DER format)
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -outform DER -out private-key.der

# Extract public key (DER format)
openssl rsa -in private-key.der -inform DER -pubout -outform DER -out public-key.der

# Base64 encode for API submission
base64 < public-key.der | tr -d '\n'
```

Then register the base64-encoded public key with the `register-key` endpoint above.
