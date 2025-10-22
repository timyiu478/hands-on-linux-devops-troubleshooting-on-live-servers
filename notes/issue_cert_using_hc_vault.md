## "Campina Grande": Give me my cert, Vault

### Problem

A web application running at https://nginx.example.com has an expired certificate. Issue a new certificate using the Hashicorp Vault running on the server.

The Vault instance is already unsealed and initialized, and you have full admin access with the admin user.

The certificate presented by Nginx is issued by the Vault PKI (check using openssl verify -CAfile /usr/local/share/ca-certificates/vault-pki-ca.crt /etc/nginx/ssl/cert.pem).

src: https://sadservers.com/scenario/campina-grande

### Solution

#### 1. list all roles configured in the PKI secrets engine

```bash
admin@i-066e07d836022be54:/etc/nginx/ssl$ vault list pki/roles
WARNING! VAULT_ADDR and -address unset. Defaulting to https://127.0.0.1:8200.
Keys
----
cert-admin
```

#### 2. Issue new cert and write it to `~/cert.json`

This command is learnt from grok.

```
admin@i-066e07d836022be54:/etc/nginx/ssl$ vault write -format json pki/issue/cert-admin common_name="nginx.example.com" ttl="24h" > ~/cert.json
```

#### 3. update `/etc/nginx/cert.pem` and `/etc/nginx/key.pem` 

```
admin@i-066e07d836022be54:/etc/nginx/ssl$ cat ~/cert.json | jq -r '.data.certificate' > cert.pem
admin@i-066e07d836022be54:/etc/nginx/ssl$ cat ~/cert.json | jq -r '.data.private_key' > key.pem
```

#### 4. restart nginx

```
admin@i-066e07d836022be54:/etc/nginx/ssl$ sudo systemctl restart nginx
admin@i-066e07d836022be54:/etc/nginx/ssl$ curl https://nginx.example.com
Hello!
```
