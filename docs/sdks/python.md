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

# List all agents
agents = client.list_agents()

# Delete an agent
client.delete_agent(agent.id)
```

## API

### `client.create_agent(name, description="") -> Agent`

Create a new agent.

### `client.list_agents() -> list[Agent]`

List all agents.

### `client.delete_agent(agent_id) -> bool`

Delete an agent.

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
    registered_since: Optional[datetime]
    error: Optional[str]
    challenge: Optional[str]
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
