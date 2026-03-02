# Node.js / TypeScript SDK

[![npm](https://img.shields.io/npm/v/tether-name)](https://www.npmjs.com/package/tether-name)

## Install

```bash
npm install tether-name
```

Requires Node.js 20+. Zero runtime dependencies — uses only Node.js built-in `crypto` and `fetch`.

## Usage

```typescript
import { TetherClient } from 'tether-name';

const client = new TetherClient({
  agentId: 'your-agent-id',
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
  // Authentication — choose one:

  // Option 1: API key (for agent management and agent operations)
  apiKey: 'sk-tether-name-...',

  // Option 2: Agent + private key (for verification and signing)
  agentId: 'your-agent-id',
  privateKeyPath: '/path/to/key.der',  // File path (DER or PEM)
  privateKeyPem: '-----BEGIN...',       // PEM string
  privateKeyBuffer: buffer,             // DER buffer
});
```

When `apiKey` is set, `agentId` and private key options become optional. A private key is still required for `verify()` and `sign()`.

Environment variables:

- `TETHER_API_KEY`
- `TETHER_AGENT_ID`
- `TETHER_PRIVATE_KEY_PATH`

## Agent Management

One line to start managing agents programmatically:

```typescript
const client = new TetherClient({ apiKey: 'sk-tether-name-...' });

// Create an agent
const agent = await client.createAgent('my-bot', 'My bot description');
console.log(agent.id);

// Create an agent with a verified domain
const domains = await client.listDomains();
const verifiedDomain = domains.find(d => d.verified);
if (verifiedDomain) {
  const agent = await client.createAgent('my-bot', 'My bot', verifiedDomain.id);
  // Verification results will show the domain instead of email
}

// List all agents
const agents = await client.listAgents();

// Delete an agent
await client.deleteAgent(agent.id);
```

## Domain Management

List domains registered to your account. Domains are claimed and verified via the [web dashboard](https://tether.name/dashboard) or [API](../api/domains.md), then referenced by ID when creating agents.

```typescript
const client = new TetherClient({ apiKey: 'sk-tether-name-...' });

// List all domains
const domains = await client.listDomains();
for (const domain of domains) {
  console.log(`${domain.domain} — ${domain.verified ? 'verified' : 'pending'}`);
}
```

## API

### `client.createAgent(name, description?, domainId?)`

Create a new agent. Optionally assign a verified domain by passing its ID.

Returns: `Promise<Agent>`

### `client.listAgents()`

List all agents.

Returns: `Promise<Agent[]>`

### `client.deleteAgent(agentId)`

Delete an agent.

Returns: `Promise<boolean>`

### `client.listDomains()`

List all registered domains for the authenticated account.

Returns: `Promise<Domain[]>`

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
  domain?: string;  // Verified domain (if assigned to this agent)
  registeredSince?: string;
  error?: string;
  challenge?: string;
}

interface Agent {
  id: string;
  agentName: string;
  description: string;
  domainId?: string;
  domain?: string | null;
  createdAt: number;
  registrationToken?: string;
  lastVerifiedAt?: number;
}

interface Domain {
  id: string;
  domain: string;
  verified: boolean;
  verifiedAt: number;
  lastCheckedAt: number;
  createdAt: number;
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
