# Credentials

Credentials represent a registered agent. Each credential has a unique ID and an associated RSA public key.

## Create a Credential

Requires authentication (JWT or API key).

```
POST /credentials/issue
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "agentName": "My Agent",
  "description": "Optional description for this agent"
}
```

**Response (201 Created):**

```json
{
  "id": "rgUOzbqar8z0Ag9RZH5I",
  "agentName": "My Agent",
  "description": "Optional description for this agent",
  "createdAt": 1736899200000,
  "registrationToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

!!! warning
    The `registrationToken` is shown only once. Save it — you'll need it to register your public key. It expires after 5 minutes.

## Register a Public Key

No authentication required — uses the registration token instead.

The `publicKey` must be a **base64-encoded DER** (SubjectPublicKeyInfo / X.509) RSA public key, minimum 2048 bits.

```
POST /credentials/{credentialId}/register-key
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

## Check Credential Status

Requires authentication (JWT or API key).

```
GET /credentials/{credentialId}/status
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

## List Credentials

Requires authentication (JWT or API key).

```
GET /credentials
Authorization: Bearer eyJ...
```

**Response:**

```json
[
  {
    "id": "rgUOzbqar8z0Ag9RZH5I",
    "agentName": "My Agent",
    "description": "Optional description",
    "createdAt": 1736899200000,
    "lastVerifiedAt": 1736985600000
  }
]
```

## Delete a Credential

Requires authentication (JWT or API key).

```
DELETE /credentials/{credentialId}
Authorization: Bearer eyJ...
```

**Response:**

```json
{
  "message": "Credential deleted"
}
```

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
