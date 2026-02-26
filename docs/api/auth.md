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

### Exchange Code for Tokens

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

### Refresh Token

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
  "accessToken": "eyJ..."
}
```

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

```
POST /auth/logout
Authorization: Bearer eyJ...
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
  "expiresAt": "2026-05-27T00:00:00.000Z",
  "createdAt": "2026-02-26T00:00:00.000Z"
}
```

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
    "expiresAt": "2026-05-27T00:00:00.000Z",
    "createdAt": "2026-02-26T00:00:00.000Z",
    "lastUsedAt": "2026-02-26T12:00:00.000Z",
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

API keys can be used with all credential endpoints (`/credentials/*`). Creating and managing API keys themselves requires JWT authentication.

### Security Notes

- API keys are hashed before storage — they cannot be retrieved after creation.
- The `sk-tether-name-` prefix enables automated leak detection in logs and repositories.
- Revoked keys are rejected immediately.
- Set `expiresInDays` to limit key lifetime. Omit for non-expiring keys.

## Public Endpoints

The following endpoints do **not** require authentication:

- `POST /challenge` — Request a challenge
- `POST /challenge/verify` — Submit proof
- `GET /check` — Check challenge status
