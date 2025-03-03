```mermaid
sequenceDiagram
    participant P as Python Script
    participant V as Vault Server

    P->>P: Import hvac library
    P->>P: Define vault_addr, vault_token, secret_path, secret_key
    P->>P: Initialize hvac.Client with vault_addr and vault_token
    activate P
    P->>V: Authenticate with token (vault_token)
    activate V
    V-->>P: Authentication successful
    deactivate V
    P->>V: Request secret version (secret_path)
    activate V
    V-->>P: Return secret data (response)
    deactivate V
    P->>P: Extract secret_value from response using secret_key
    P->>P: Print secret_value
    deactivate P
