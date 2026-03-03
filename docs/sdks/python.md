# Python SDK

[![PyPI](https://img.shields.io/pypi/v/tether-name)](https://pypi.org/project/tether-name/)

## Install

```bash
pip install tether-name
```

Requires Python 3.8+.

## Usage

```python
from tether_name import TetherClient

client = TetherClient(
    agent_id="your-agent-id",
    private_key_path="/path/to/private-key.der"
)

# Full verification in one call
result = client.verify()

# Or step by step
challenge = client.request_challenge()
proof = client.sign(challenge)
result = client.submit_proof(challenge, proof)
```

## Configuration

```python
client = TetherClient(
    # Authentication — choose one:

    # Option 1: API key (for agent management and agent operations)
    api_key="sk-tether-name-...",

    # Option 2: Agent + private key (for verification and signing)
    agent_id="your-agent-id",
    private_key_path="/path/to/key.der",     # File path (DER or PEM)
    private_key_pem="-----BEGIN...",          # PEM string or bytes
    private_key_der=b"...",                   # DER bytes
)
```

When `api_key` is set, `agent_id` and private key options become optional. A private key is still required for `verify()` and `sign()`.

Environment variables:

- `TETHER_API_KEY`
- `TETHER_AGENT_ID`
- `TETHER_PRIVATE_KEY_PATH`

## Context Manager

```python
with TetherClient(agent_id="...", private_key_path="...") as client:
    result = client.verify()
# HTTP client automatically closed
```

## Agent Management

One line to start managing agents programmatically:

```python
client = TetherClient(api_key="sk-tether-name-...")

# Create an agent
agent = client.create_agent("my-bot", "My bot description")
print(agent.id)

# Create an agent with a verified domain
domains = client.list_domains()
verified = [d for d in domains if d.verified]
if verified:
    agent = client.create_agent("my-bot", "My bot", domain_id=verified[0].id)
    # Verification results will show the domain instead of email

# List all agents
agents = client.list_agents()

# Delete an agent
client.delete_agent(agent.id)

# Key lifecycle operations
keys = client.list_agent_keys(agent.id)
rotated = client.rotate_agent_key(
    agent.id,
    public_key="BASE64_SPKI_PUBLIC_KEY",
    grace_period_hours=24,
    reason="routine_rotation",
    step_up_code="123456",  # or challenge+proof
)
client.revoke_agent_key(
    agent.id,
    rotated.new_key_id,
    reason="compromised",
    step_up_code="654321",  # or challenge+proof
)
```

## Domain Management

List domains registered to your account. Domains are claimed and verified via the [web dashboard](https://tether.name/dashboard) or [API](../api/domains.md), then referenced by ID when creating agents.

```python
client = TetherClient(api_key="sk-tether-name-...")

# List all domains
domains = client.list_domains()
for domain in domains:
    status = "verified" if domain.verified else "pending"
    print(f"{domain.domain} — {status}")
```

## API

### `client.create_agent(name, description="", domain_id="") -> Agent`

Create a new agent. Optionally assign a verified domain by passing its ID.

### `client.list_agents() -> list[Agent]`

List all agents.

### `client.delete_agent(agent_id) -> bool`

Delete an agent.

### `client.list_domains() -> list[Domain]`

List all registered domains for the authenticated account.

### `client.list_agent_keys(agent_id) -> list[AgentKey]`

List key lifecycle entries (`active`, `grace`, `revoked`) for an agent.

### `client.rotate_agent_key(...) -> RotateKeyResult`

Rotate an agent key. Requires API key auth plus step-up verification via either `step_up_code` or `challenge` + `proof`.

### `client.revoke_agent_key(...) -> RevokeKeyResult`

Revoke an agent key. Requires API key auth plus step-up verification via either `step_up_code` or `challenge` + `proof`.

### `client.verify() -> VerificationResult`

Complete verification flow. Requests a challenge, signs it, and submits proof.

### `client.request_challenge() -> str`

Request a new challenge from the Tether API.

### `client.sign(challenge) -> str`

Sign a challenge string with the configured private key.

Returns: URL-safe base64 signature (no padding).

### `client.submit_proof(challenge, proof) -> VerificationResult`

Submit a signed proof for verification.

### `client.close()`

Close the HTTP client. Called automatically when using context manager.

## Types

```python
@dataclass
class VerificationResult:
    verified: bool
    agent_name: Optional[str]
    verify_url: Optional[str]
    email: Optional[str]
    domain: Optional[str]  # Verified domain (if assigned to this agent)
    registered_since: Optional[datetime]
    error: Optional[str]
    challenge: Optional[str]

@dataclass
class Agent:
    id: str
    agent_name: str
    description: str
    domain_id: str
    domain: Optional[str]
    created_at: int  # epoch ms
    registration_token: str
    last_verified_at: int

@dataclass
class Domain:
    id: str
    domain: str
    verified: bool
    verified_at: int
    last_checked_at: int
    created_at: int

@dataclass
class AgentKey:
    id: str
    status: str
    created_at: int
    activated_at: int
    grace_until: int
    revoked_at: int
    revoked_reason: str = ""
```

## Error Handling

```python
from tether_name import (
    TetherError,              # Base exception
    TetherAPIError,           # API request failures
    TetherVerificationError,  # Verification failures
    TetherKeyError,           # Private key issues
)

try:
    result = client.verify()
except TetherAPIError as e:
    print(f"API error {e.status_code}: {e.message}")
except TetherVerificationError as e:
    print(f"Verification failed: {e.message}")
except TetherKeyError as e:
    print(f"Key error: {e.message}")
```

## Dependencies

- `httpx` >= 0.20.0
- `cryptography` >= 3.4.0

## Links

- [PyPI](https://pypi.org/project/tether-name/)
- [GitHub](https://github.com/tether-name/tether-name-python)
