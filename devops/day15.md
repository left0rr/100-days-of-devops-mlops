# DevOps Day 15 — Nginx & SSL/TLS Configuration

## 1. Install and enable Nginx

On RHEL/CentOS-based systems:

```bash
sudo yum install -y nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

- **Nginx:** High-performance web server, reverse proxy, and load balancer.
- `enable`: Starts Nginx automatically at boot.
- `--now`: Starts it immediately.

## 2. Nginx configuration structure

Main configuration:

```text
/etc/nginx/nginx.conf
```

Additional server configurations:

```text
/etc/nginx/conf.d/*.conf
```

Default document root on this server:

```text
/usr/share/nginx/html
```

Default index file:

```text
index.html
```

Useful commands:

```bash
sudo nginx -t                 # Validate configuration syntax
sudo nginx -T                 # Display complete loaded configuration
sudo systemctl reload nginx   # Apply changes without full restart
sudo systemctl restart nginx  # Restart service
```

**Important:** Always run `nginx -t` before reloading or restarting after configuration changes.

## 3. SSL/TLS certificates and private keys

TLS encrypts communication between clients and servers, protecting transmitted data and authenticating the server.

Important files:
- `.crt` — Public certificate containing the server identity and public key.
- `.key` — Private key used by the server during TLS authentication.
- `.csr` — Certificate Signing Request sent to a Certificate Authority (CA).

Common certificate locations:

```text
/etc/nginx/ssl/
/etc/pki/tls/certs/
/etc/pki/tls/private/
```

Create a dedicated directory and move certificates:

```bash
sudo mkdir -p /etc/nginx/ssl
sudo mv server.crt /etc/nginx/ssl/
sudo mv server.key /etc/nginx/ssl/
```

Protect the private key:

```bash
sudo chmod 644 /etc/nginx/ssl/server.crt
sudo chmod 600 /etc/nginx/ssl/server.key
```

**Security principles:**
- Never expose or commit private keys to Git.
- Restrict private-key access to authorized users and services.
- Use trusted CA-issued certificates in production.
- Self-signed certificates are useful for labs and internal testing but are not automatically trusted by clients.

## 4. Configure an HTTPS server block

Example configuration in `/etc/nginx/conf.d/https.conf`:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name example.com;

    root /usr/share/nginx/html;
    index index.html;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Important directives:

| Directive | Purpose |
|---|---|
| `listen 443 ssl` | Enables HTTPS on port 443 |
| `server_name` | Defines the hostname handled by this server |
| `root` | Defines the website document root |
| `index` | Specifies the default page |
| `ssl_certificate` | Points to the public certificate |
| `ssl_certificate_key` | Points to the private key |
| `location` | Defines how matching requests are handled |
| `try_files` | Checks requested files and returns 404 if missing |

HTTP normally uses port **80**; HTTPS normally uses port **443**.

## 5. Deploy website content

```bash
echo 'Welcome!' | sudo tee /usr/share/nginx/html/index.html
```

Nginx serves files from its configured document root.

Ensure the Nginx worker process has permission to read the website files.

## 6. Validate and test HTTPS

Validate and apply configuration:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Test from another machine:

```bash
curl -Ik https://example.com/
```

Retrieve the response body:

```bash
curl -k https://example.com/
```

Useful curl flags:

| Flag | Meaning |
|---|---|
| `-I` | Fetch response headers only |
| `-k` | Skip certificate verification |
| `-v` | Display detailed connection and TLS debugging information |
| `-L` | Follow HTTP redirects |

**Important:** `-k` is appropriate for testing self-signed certificates, not a production security solution.

Inspect certificate details:

```bash
openssl x509 -in server.crt -noout -subject -issuer -dates
```

Inspect certificate and private-key consistency:

```bash
openssl x509 -in server.crt -pubkey -noout | openssl sha256
openssl pkey -in server.key -pubout | openssl sha256
```

The two hashes should match.

## 7. Troubleshooting checklist

| Symptom | What to investigate |
|---|---|
| Nginx fails to start | `sudo nginx -t` and `journalctl -u nginx` |
| HTTPS connection refused | Nginx listener, port 443, firewall |
| 403 Forbidden | File permissions, directory traversal permissions, SELinux |
| 404 Not Found | Document root, requested path, `index` |
| Certificate error | Certificate validity, hostname/SAN, CA trust, certificate chain |
| Private-key error | Key path, permissions, certificate/key mismatch |

Useful diagnostic commands:

```bash
sudo ss -lntp
sudo journalctl -u nginx -xe
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

If `ss` is unavailable, check whether `iproute` is installed or use another available diagnostic tool.

## 8. Production concepts to learn next

- **Reverse proxy:** Nginx receives client requests and forwards them to backend applications.
- **Load balancing:** Distributes requests across multiple backend servers.
- **TLS termination:** Nginx handles HTTPS encryption and forwards requests to backend services.
- **Certificate renewal:** Production certificates need automated renewal and reload procedures.
- **HTTP-to-HTTPS redirect:** Redirects port 80 traffic to HTTPS.
- **Security headers:** Examples include HSTS, X-Content-Type-Options, and Content-Security-Policy.
- **SELinux:** Can restrict Nginx access even when ordinary Linux permissions appear correct.

**DevOps/MLOps takeaway:** Nginx is commonly used as the secure entry point for web applications, APIs, model-serving endpoints, and inference services. Knowing how to configure HTTPS, protect private keys, validate configurations, and troubleshoot connectivity is a valuable production skill.