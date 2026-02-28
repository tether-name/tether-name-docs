# CLI

[![npm](https://img.shields.io/npm/v/tether-name-cli)](https://www.npmjs.com/package/tether-name-cli)

The Tether CLI lets you set up credentials, verify your agent's identity, and debug the challenge-response flow — all from the terminal.

## Install

```bash
npm install -g tether-name-cli
```

Or use without installing:

```bash
npx tether-name-cli verify
```

## Commands

### `tether init`

Interactive setup wizard. Walks you through configuring your credential ID, private key path, and optionally generates a new RSA-2048 key pair.

```bash
tether init
```

This saves your configuration to `~/.tether/config.json`.

If you choose to generate keys, it creates:

- `.tether-private-key.pem` (private key, `chmod 600`)
- `.tether-public-key.pem` (public key, for registering with Tether)

### `tether verify`

Perform a full identity verification — requests a challenge, signs it, submits proof, and displays the result.

```bash
tether verify
```

```bash
# Machine-readable output
tether verify --json
```

### `tether status`

Show your current configuration — credential ID (masked), key file path, and API URL.

```bash
tether status
```

```bash
tether status --json
```

### `tether challenge`

Request a new challenge code from the Tether API and print it.

```bash
tether challenge
```

### `tether sign <challenge>`

Sign a challenge string with your private key and print the proof.

```bash
tether sign "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

### `tether check <code>`

Check the status of a challenge by its code.

```bash
tether check "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

```bash
tether check "a1b2c3d4-e5f6-7890-abcd-ef1234567890" --json
```

### `tether agent create <name>`

Create a new agent. Requires an API key.

```bash
tether agent create "my-bot" --description "My helpful bot"
```

```bash
tether agent create "my-bot" --json
```

Returns the agent's ID, name, and a **registration token** (save it — it can't be retrieved later).

### `tether agent list`

List all agents associated with your API key.

```bash
tether agent list
```

```bash
tether agent list --json
```

### `tether agent delete <id>`

Delete an agent by its credential ID.

```bash
tether agent delete "credential-id-here"
```

```bash
tether agent delete "credential-id-here" --json
```

## Configuration

The CLI resolves configuration in this order (first wins):

1. **CLI flags** — `--credential-id`, `--key-path`, `--api-key`
2. **Environment variables** — `TETHER_CREDENTIAL_ID`, `TETHER_PRIVATE_KEY_PATH`, `TETHER_API_KEY`
3. **Config file** — `~/.tether/config.json` (created by `tether init`)

### Global Flags

| Flag | Description |
|---|---|
| `--credential-id <id>` | Override credential ID |
| `--key-path <path>` | Override private key file path |
| `--api-key <key>` | API key for agent management operations |
| `--verbose` | Enable debug output |
| `--json` | Machine-readable JSON output (on supported commands) |

## Example Workflows

### Verification

```bash
# 1. Set up credentials
tether init

# 2. Check your config
tether status

# 3. Verify your identity
tether verify

# 4. Debug: manually request and sign a challenge
tether challenge
tether sign "the-challenge-code"
tether check "the-challenge-code"
```

### Agent Management

```bash
# Create an agent
tether agent create "my-bot" --description "My helpful assistant"

# List all agents
tether agent list

# Delete an agent
tether agent delete "credential-id"
```

Agent management commands require an API key. Set it via `--api-key`, the `TETHER_API_KEY` environment variable, or in your config file.

## Links

- [npm](https://www.npmjs.com/package/tether-name-cli)
- [GitHub](https://github.com/tether-name/tether-name-cli)
