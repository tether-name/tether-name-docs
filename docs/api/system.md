# System & Metadata Endpoints

These endpoints provide health checks, machine-readable metadata, and operational/admin utilities.

## Health

### `GET /`

Simple liveness check.

**Response:**

```text
OK
```

## Agent/Protocol Metadata

### `GET /AGENTS.md`

Returns agent-readable instructions (Markdown) for verification behavior.

### `GET /.well-known/tether-name.json`

Returns machine-readable protocol metadata and endpoint map.

**Example response:**

```json
{
  "name": "tether.name",
  "version": "2.0",
  "protocol": "tether-name-challenge",
  "api_url": "https://api.tether.name",
  "endpoints": {
    "generate_challenge": "POST /challenge",
    "verify_challenge": "POST /challenge/verify"
  }
}
```

## Build & Global Stats

### `GET /version`

Returns service build metadata.

**Response:**

```json
{
  "buildTimestamp": "1741111111111",
  "buildTimestampPretty": "Tuesday, March 4, 2026 at 6:11:11 PM CST"
}
```

### `GET /stats`

Returns global counters.

**Response:**

```json
{
  "totalVerifications": 12345,
  "totalAgentsRegistered": 2345,
  "totalDomainsVerified": 678
}
```

## Admin Endpoints

Admin routes require allowlisted admin auth and are intended for operational automation.

### `DELETE /admin/cleanup`

Deletes expired auth artifacts.

**Response:**

```json
{
  "authCodesDeleted": 10,
  "authCodeExchangeTokensDeleted": 4,
  "challengesDeleted": 21,
  "refreshTokensDeleted": 7
}
```

### `POST /admin/domain-reverify`

Re-checks verified domains and revokes those that no longer have the required DNS TXT record.

**Response:**

```json
{
  "checked": 3,
  "revoked": 1,
  "stillValid": 2,
  "errors": 0,
  "details": [
    {
      "domainId": "abc123",
      "domain": "example.com",
      "userId": "user1",
      "status": "valid"
    }
  ]
}
```

### Admin auth configuration

Preferred admin config:

- `ADMIN_USER_IDS` (comma-separated user IDs)
- and/or `ADMIN_EMAILS` (comma-separated emails)
- optional `ADMIN_ALLOW_API_KEYS=true` to allow API key auth for allowlisted admins (default: false)

For Cloud Scheduler / Cloud Tasks, you can also allow Google OIDC service-account tokens:

- `ADMIN_OIDC_SERVICE_ACCOUNTS`
- optional `ADMIN_OIDC_AUDIENCE`

Use:

```
Authorization: Bearer <jwt-or-api-key-or-google-oidc-token>
```

Legacy fallback is still supported via `ADMIN_SECRET` (`Authorization: Bearer <secret>`).
