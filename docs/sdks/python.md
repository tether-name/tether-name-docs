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
    credential_id="your-credential-id",
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
    # Required
    credential_id="your-credential-id",

    # Private key — provide one of:
    private_key_path="/path/to/key.der",     # File path (DER or PEM)
    private_key_pem="-----BEGIN...",          # PEM string or bytes
    private_key_der=b"...",                   # DER bytes

    # Optional
    base_url="https://api.tether.name",      # API base URL
    timeout=30.0                              # Request timeout in seconds
)
```

All options fall back to environment variables:

- `TETHER_CREDENTIAL_ID`
- `TETHER_PRIVATE_KEY_PATH`

## Context Manager

```python
with TetherClient(credential_id="...", private_key_path="...") as client:
    result = client.verify()
# HTTP client automatically closed
```

## API

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
