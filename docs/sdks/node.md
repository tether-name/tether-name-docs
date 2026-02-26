# Node.js / TypeScript SDK

[![npm](https://img.shields.io/npm/v/tether-name)](https://www.npmjs.com/package/tether-name)

## Install

```bash
npm install tether-name
```

Requires Node.js 18+. Zero runtime dependencies — uses only Node.js built-in `crypto` and `fetch`.

## Usage

```typescript
import { TetherClient } from 'tether-name';

const client = new TetherClient({
  credentialId: 'your-credential-id',
  privateKeyPath: '/path/to/private-key.der'
});

// Full verification in one call
const result = await client.verify();

// Or step by step
const challenge = await client.requestChallenge();
const proof = client.sign(challenge);
const result = await client.submitProof(challenge, proof);
```

## Configuration

```typescript
const client = new TetherClient({
  // Required
  credentialId: 'your-credential-id',

  // Private key — provide one of:
  privateKeyPath: '/path/to/key.der',  // File path (DER or PEM)
  privateKeyPem: '-----BEGIN...',       // PEM string
  privateKeyBuffer: buffer,             // DER buffer

  // Optional
  baseUrl: 'https://api.tether.name'   // API base URL
});
```

Credential ID and key path can also be set via environment variables:

- `TETHER_CREDENTIAL_ID`
- `TETHER_PRIVATE_KEY_PATH`

## API

### `client.verify()`

Complete verification flow. Requests a challenge, signs it, and submits proof.

Returns: `Promise<VerificationResult>`

### `client.requestChallenge()`

Request a new challenge from the Tether API.

Returns: `Promise<string>`

### `client.sign(challenge)`

Sign a challenge string with the configured private key.

Returns: `string` (URL-safe base64 signature)

### `client.submitProof(challenge, proof)`

Submit a signed proof for verification.

Returns: `Promise<VerificationResult>`

## Types

```typescript
interface VerificationResult {
  verified: boolean;
  agentName?: string;
  verifyUrl?: string;
  email?: string;
  registeredSince?: string;
  error?: string;
  challenge?: string;
}
```

## Error Handling

```typescript
import { TetherError, TetherAPIError, TetherVerificationError } from 'tether-name';

try {
  const result = await client.verify();
} catch (error) {
  if (error instanceof TetherAPIError) {
    console.error(`API error: ${error.status} - ${error.message}`);
  } else if (error instanceof TetherVerificationError) {
    console.error(`Verification failed: ${error.message}`);
  }
}
```

## Links

- [npm](https://www.npmjs.com/package/tether-name)
- [GitHub](https://github.com/tether-name/tether-name-node)
