# Credentials

Credentials represent a registered agent. Each credential has a unique ID and an associated RSA public key.

## Create a Credential

Requires authentication.

```
POST /credentials/issue
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "name": "My Agent"
}
```

**Response:**

```json
{
  "credentialId": "rgUOzbqar8z0Ag9RZH5I",
  "registrationToken": "one-time-token"
}
```

!!! warning
    The `registrationToken` is shown only once. Save it — you'll need it to register your public key.

## Register a Public Key

No authentication required — uses the registration token instead.

```
POST /credentials/{credentialId}/register-key
Content-Type: application/json

{
  "registrationToken": "one-time-token",
  "publicKey": "-----BEGIN PUBLIC KEY-----\n..."
}
```

**Response:**

```json
{
  "registered": true
}
```

## Check Credential Status

Requires authentication.

```
GET /credentials/{credentialId}/status
Authorization: Bearer eyJ...
```

**Response:**

```json
{
  "registered": true
}
```

## List Credentials

Requires authentication.

```
GET /credentials
Authorization: Bearer eyJ...
```

Returns an array of all credentials associated with your account.

## Delete a Credential

Requires authentication.

```
DELETE /credentials/{credentialId}
Authorization: Bearer eyJ...
```

## Key Generation

Generate an RSA-2048 key pair for your agent:

```bash
# Generate private key (DER format)
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -outform DER -out private-key.der

# Extract public key (PEM format for registration)
openssl rsa -in private-key.der -inform DER -pubout -out public-key.pem
```

Then register the public key contents with the endpoint above.
