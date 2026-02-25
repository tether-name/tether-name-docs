# Register Your Agent

## 1. Create an Account

Go to [tether.name/auth](https://tether.name/auth) and enter your email. You'll receive a magic code — no passwords.

## 2. Create a Credential

From the dashboard, click **New Agent** and give it a name. You'll receive:

- A **Credential ID**
- A **Registration Token**

Save both — the registration token is shown only once.

## 3. Generate and Register Keys

Your agent needs to generate an RSA-2048 key pair and register the public key with Tether.

=== "Node.js"

    ```typescript
    import { TetherClient } from 'tether-name';

    const client = new TetherClient({
      credentialId: 'your-credential-id',
      privateKeyPath: '/path/to/private-key.der'
    });
    ```

=== "Python"

    ```python
    from tether_name import TetherClient

    client = TetherClient(
        credential_id="your-credential-id",
        private_key_path="/path/to/private-key.der"
    )
    ```

=== "Go"

    ```go
    import tether "github.com/tether-name/tether-name-go"

    client, err := tether.NewClient(tether.Config{
        CredentialID:   "your-credential-id",
        PrivateKeyPath: "/path/to/private-key.der",
    })
    ```

## 4. Store Your Keys Safely

- Keep the private key file secure — treat it like an SSH key
- The key should be readable only by the agent process
- Never commit private keys to version control
- Add `*.der` and `*.pem` to your `.gitignore`
