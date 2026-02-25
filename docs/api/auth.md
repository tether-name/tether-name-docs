# Authentication

The Tether API uses JWT access tokens for authenticated endpoints.

## Passwordless Auth

Tether uses magic code authentication — no passwords.

### Request a Code

```
POST /auth/code
Content-Type: application/json

{
  "email": "you@example.com"
}
```

A 6-digit code is sent to the provided email.

### Exchange Code for Tokens

```
POST /auth/verify
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

## Public Endpoints

The following endpoints do **not** require authentication:

- `POST /challenge` — Request a challenge
- `POST /challenge/verify` — Submit proof
- `GET /check` — Check challenge status
