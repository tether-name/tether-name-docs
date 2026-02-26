# Authentication

The Tether API uses JWT access tokens for authenticated endpoints.

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

## Public Endpoints

The following endpoints do **not** require authentication:

- `POST /challenge` — Request a challenge
- `POST /challenge/verify` — Submit proof
- `GET /check` — Check challenge status
