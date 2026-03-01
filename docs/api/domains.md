# Domains

Domain verification lets organizations prove domain ownership via DNS, showing their domain on verification results instead of an email address.

## Claim a Domain

Requires authentication (JWT or API key).

```
POST /domains/claim
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "domain": "example.com"
}
```

**Response (201 Created):**

```json
{
  "id": "abc123def456",
  "domain": "example.com",
  "txtRecord": "tether-verify=a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "txtHost": "_tether.example.com",
  "instructions": "Add a TXT record for _tether.example.com with value: tether-verify=a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

!!! info
    If you've already claimed this domain, the existing claim is returned (200 OK) with the same DNS instructions.

### DNS Setup

Add a **TXT record** to your domain's DNS with the values from the response:

| Field | Value |
|-------|-------|
| **Type** | `TXT` |
| **Host/Name** | `_tether.example.com` |
| **Value** | `tether-verify=<your token>` |

!!! tip
    Most DNS providers update within a few minutes, but propagation can take up to 48 hours. You can verify at any time from your dashboard or via the API.

## Verify a Domain

Triggers a DNS lookup to check for the TXT record. Requires authentication.

```
POST /domains/{domainId}/verify
Authorization: Bearer eyJ...
```

**Response (verified):**

```json
{
  "verified": true,
  "domain": "example.com",
  "message": "Domain verified successfully"
}
```

**Response (not yet verified):**

```json
{
  "verified": false,
  "domain": "example.com",
  "message": "DNS record not found. Add a TXT record for _tether.example.com with value: tether-verify=...",
  "txtHost": "_tether.example.com",
  "txtRecord": "tether-verify=a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

## List Domains

Requires authentication (JWT or API key).

```
GET /domains
Authorization: Bearer eyJ...
```

**Response:**

```json
[
  {
    "id": "abc123def456",
    "domain": "example.com",
    "verified": true,
    "verifiedAt": 1736899200000,
    "lastCheckedAt": 1736899200000,
    "createdAt": 1736898000000
  }
]
```

## Delete a Domain

Removes a domain claim. Requires authentication.

```
DELETE /domains/{domainId}
Authorization: Bearer eyJ...
```

**Response:**

```json
{
  "message": "Domain removed"
}
```

## How It Affects Verification

Once a domain is verified, the [challenge verify response](challenges.md#verify-a-challenge) and [challenge status](challenges.md#check-challenge-status) will include `domain` instead of `email`:

```json
{
  "valid": true,
  "verifyUrl": "https://tether.name/check?challenge=...",
  "agentName": "My Agent",
  "domain": "example.com",
  "registeredSince": 1736899200000
}
```

This applies to **all agents** under the account with the verified domain.

## Limits

| Limit | Value |
|-------|-------|
| Domains per account | 5 |
| Domain name length | 253 characters |
| Rate limit (CRUD) | 10 requests/minute |
| Rate limit (verify) | 5 requests/minute |
