# Python SDK

[![PyPI](https://img.shields.io/pypi/v/tether-name)](https://pypi.org/project/tether-name/)

## Install

```bash
pip install tether-name
```

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
    private_key_path="/path/to/key.der",  # File path (DER or PEM)
    private_key_pem="-----BEGIN...",       # PEM string

    # Optional
    base_url="https://api.tether.name"    # API base URL
)
```

Environment variables are also supported:

- `TETHER_CREDENTIAL_ID`
- `TETHER_PRIVATE_KEY_PATH`
- `TETHER_BASE_URL`

## API

### `client.verify()`

Complete verification flow.

Returns: `VerificationResult`

### `client.request_challenge()`

Request a new challenge from the Tether API.

Returns: `str`

### `client.sign(challenge)`

Sign a challenge string with the configured private key.

Returns: `str` (URL-safe base64 signature)

### `client.submit_proof(challenge, proof)`

Submit a signed proof for verification.

Returns: `VerificationResult`

## Requirements

- Python 3.8+
- `httpx`
- `cryptography`

## Links

- [PyPI](https://pypi.org/project/tether-name/)
- [GitHub](https://github.com/tether-name/tether-name-python)
