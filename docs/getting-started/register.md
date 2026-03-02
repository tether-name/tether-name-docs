# Register Your Agent

## 1. Create an Account

Go to [tether.name/auth](https://tether.name/auth) and enter your email. You'll receive a magic code — no passwords.

## 2. Create an Agent

From the dashboard, click **New Agent** and give it a name. You'll receive:

- A **Agent ID**
- A **Registration Token**

Save both — the registration token is shown only once.

## 3. Generate and Register Keys

Your agent needs to generate an RSA-2048 key pair and register the **public key** with Tether.

### Generate keypair (once)

```bash
# 1) Generate private key (DER format)
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -outform DER -out private-key.der

# 2) Export public key (DER, SubjectPublicKeyInfo)
openssl rsa -in private-key.der -inform DER -pubout -outform DER -out public-key.der

# 3) Base64 encode for API submission
PUBLIC_KEY_BASE64="$(base64 < public-key.der | tr -d '\n')"
```

### Register public key

```bash
curl -X POST "https://api.tether.name/agents/{agentId}/register-key" \
  -H "Content-Type: application/json" \
  -d "{\"registrationToken\":\"{registrationToken}\",\"publicKey\":\"$PUBLIC_KEY_BASE64\"}"
```

Expected response:

```json
{
  "registered": true
}
```

### Configure your SDK with the private key

=== "Node.js"

    ```typescript
    import { TetherClient } from 'tether-name';

    const client = new TetherClient({
      agentId: 'your-agent-id',
      privateKeyPath: '/path/to/private-key.der'
    });
    ```

=== "Python"

    ```python
    from tether_name import TetherClient

    client = TetherClient(
        agent_id="your-agent-id",
        private_key_path="/path/to/private-key.der"
    )
    ```

=== "Go"

    ```go
    import tether "github.com/tether-name/tether-name-go/v2"

    client, err := tether.NewClient(tether.Options{
        AgentID:   "your-agent-id",
        PrivateKeyPath: "/path/to/private-key.der",
    })
    ```

## 4. Store Your Keys Safely

- Keep the private key file secure — treat it like an SSH key
- The key should be readable only by the agent process
- Never commit private keys to version control
- Add `*.der` and `*.pem` to your `.gitignore`
