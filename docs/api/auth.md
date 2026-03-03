# Authentication

The Tether API supports two authentication methods: JWT access tokens and API keys.

## Passwordless Auth

Tether uses magic code authentication — no passwords.

### Request a Code

```
POST /auth/send-code
Content-Type: application/json

{
  "email": "you@example.com"
}
```

A magic code is sent to the provided email.

### Exchange Code for Tokens (manual entry flow)

```
POST /auth/verify-code
Content-Type: application/json

{
  "email": "you@example.com",
  "code": "123456"
}
```

**Response:**

```json
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ..."
}
```

### Exchange One-Time Magic Link Token

When users click the magic link from email, clients can exchange the opaque token directly (without exposing email/code in the URL):

```
POST /auth/exchange-code
Content-Type: application/json

{
  "token": "opaque-one-time-token"
}
```

**Response:**

```json
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ..."
}
```

### Refresh Token (body mode)

```
POST /auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJ..."
}
```

**Response:**

```json
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ..."
}
```

### Browser Cookie Session Mode (recommended for web apps)

For browser clients, use cookie mode so refresh tokens are never readable by JavaScript:

- Send header `X-Tether-Session-Mode: cookie` on:
  - `POST /auth/verify-code`
  - `POST /auth/exchange-code`
  - `POST /auth/refresh`
  - `POST /auth/logout`
- Include credentials in browser requests.
- API sets `tether_refresh_token` as an httpOnly cookie.
- In cookie mode, `refreshToken` in JSON responses is blank (use cookie rotation instead).

Cookie-mode refresh call:

```
POST /auth/refresh
X-Tether-Session-Mode: cookie
```

!!! note
    Refresh tokens are rotated on each use — the old token is invalidated and a new pair is returned. Always store and use the new `refreshToken` from the response.

## Using Access Tokens

Include the access token in the `Authorization` header:

```
Authorization: Bearer eyJ...
```

### Get Current User

```
GET /auth/me
Authorization: Bearer eyJ...
```

Returns the currently authenticated user's info.

### Logout

Body mode:

```
POST /auth/logout
Content-Type: application/json

{
  "refreshToken": "eyJ..."
}
```

Cookie mode:

```
POST /auth/logout
X-Tether-Session-Mode: cookie
```

Invalidates the current session.

## API Keys

API keys are long-lived tokens for programmatic access. Use them to manage agents from CI/CD pipelines, scripts, or backend services without going through the magic code flow.

API keys use the `sk-tether-name-` prefix for easy identification and leak detection.

### Create an API Key

Requires JWT authentication.

```
POST /api-keys
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "name": "CI Pipeline",
  "expiresInDays": 90
}
```

**Response:**

```json
{
  "id": "key_abc123",
  "key": "sk-tether-name-...",
  "name": "CI Pipeline",
  "keyPrefix": "sk-tether-name-abc",
  "expiresAt": 1748304000000,
  "createdAt": 1740528000000
}
```

!!! note
    Timestamps are Unix epoch milliseconds. `expiresAt` is `0` if the key doesn't expire.

!!! warning
    The full `key` value is shown only once. Store it securely — the API stores only a hash.

### List API Keys

```
GET /api-keys
Authorization: Bearer eyJ...
```

**Response:**

```json
[
  {
    "id": "key_abc123",
    "name": "CI Pipeline",
    "keyPrefix": "sk-tether-name-abc",
    "expiresAt": 1748304000000,
    "createdAt": 1740528000000,
    "lastUsedAt": 1740571200000,
    "revoked": false
  }
]
```

### Revoke an API Key

```
DELETE /api-keys/{id}
Authorization: Bearer eyJ...
```

### Using API Keys

Include the API key in the `Authorization` header, just like a JWT:

```
Authorization: Bearer sk-tether-name-...
```

API keys can be used with most agent endpoints (`/agents/*`) and domain endpoints (`/domains/*`). Creating and managing API keys themselves requires JWT authentication.

!!! warning
    Key lifecycle endpoints (`GET /agents/{id}/keys`, `POST /agents/{id}/keys/rotate`, `POST /agents/{id}/keys/{keyId}/revoke`) require a JWT access token plus step-up verification where applicable.

### Limits

| Limit | Value |
|-------|-------|
| API keys per account | 10 |
| Rate limit | 10 requests/minute |

!!! info
    If you need more API keys, please contact us at [support@tether.name](mailto:support@tether.name).

### Security Notes

- API keys are hashed before storage — they cannot be retrieved after creation.
- The `sk-tether-name-` prefix enables automated leak detection in logs and repositories.
- Revoked keys are rejected immediately.
- Set `expiresInDays` to limit key lifetime. Omit for non-expiring keys.

## Public Endpoints

The following endpoints do **not** require authentication:

- `POST /challenge` — Request a challenge
- `POST /challenge/verify` — Submit proof
- `GET /challenge/{code}` — Check challenge status
- `GET /.well-known/tether-name.json` — Machine-readable endpoint map and protocol metadata
