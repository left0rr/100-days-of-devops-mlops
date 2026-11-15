# DevOps Day 16 — Nginx Load Balancer

## 1. Challenge objective

Configure Nginx on `stlb01` to distribute HTTP traffic across three Apache application servers.

```text
              Client / Jump Host
                      |
                   HTTP :80
                      |
                 Nginx (LBR)
                   stlb01
                 /    |    \
                /     |     \
             App 1  App 2  App 3
            Apache Apache Apache
            :5004  :5004  :5004
```

**Key idea:** Clients connect to one endpoint (Nginx), which forwards requests to the backend servers.

## 2. Core concepts

- **Load balancer:** Distributes incoming requests across multiple servers to improve scalability, availability, and performance.
- **Reverse proxy:** Receives client requests and forwards them to backend services, hiding internal server details.
- **Upstream:** An Nginx group containing backend servers.
- **Proxy pass:** Tells Nginx where to forward incoming requests.

### Load-balancing algorithms

| Algorithm | Behavior |
|---|---|
| Round robin | Distributes requests sequentially (Nginx default) |
| Least connections | Selects the server with the fewest active connections |
| IP hash | Uses client IP to select a backend (basic session affinity) |

## 3. Nginx configuration

Main configuration file:

```bash
/etc/nginx/nginx.conf
```

**Configuration hierarchy:**
- `http`: HTTP settings and backend groups.
- `upstream`: Defines backend servers.
- `server`: Defines the frontend listener.
- `location`: Defines request routing rules.

### Backend pool — inside `http {}`, outside `server {}`

```nginx
upstream app_servers {
    server 10.244.234.242:5004;
    server 10.244.240.159:5004;
    server 10.244.189.207:5004;
}
```

### Frontend routing — inside the existing `server {}`

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://app_servers;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Important directives:**

| Directive | Purpose |
|---|---|
| `listen 80` | Accepts HTTP traffic on port 80 |
| `upstream` | Defines backend pool |
| `location /` | Matches incoming URL paths |
| `proxy_pass` | Forwards requests to backend pool |
| `X-Real-IP` | Passes client IP |
| `X-Forwarded-For` | Preserves proxy/client IP chain |
| `X-Forwarded-Proto` | Passes original HTTP/HTTPS scheme |

Forwarded headers help backend applications identify the original client and request protocol.

## 4. Essential commands

### Service management

```bash
sudo yum install -y nginx
sudo systemctl enable --now nginx
sudo systemctl status nginx
sudo systemctl status httpd
```

### Configuration

```bash
sudo vi /etc/nginx/nginx.conf
sudo nginx -t
sudo systemctl reload nginx
```

- `nginx -t`: Validates configuration syntax and referenced files.
- `reload`: Applies valid configuration changes without a full restart.
- Always test configuration before reloading.

### Network troubleshooting

```bash
ip addr
sudo ss -lntp
sudo ss -lntp | grep 5004
```

Check Apache's configured port:

```bash
sudo grep -R '^[[:space:]]*Listen' /etc/httpd/conf /etc/httpd/conf.d/
```

Test each backend directly from the LBR:

```bash
curl http://BACKEND_IP:PORT
```

Test through the load balancer from the jump host:

```bash
curl http://stlb01:80
```

### Logs

```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

- Access log: incoming requests and HTTP responses.
- Error log: configuration and upstream connection problems.

## 5. Troubleshooting workflow

```text
1. Discover current backend IPs and ports
                 |
2. Check Apache service status
                 |
3. Verify listening ports (ss -lntp)
                 |
4. Curl each backend directly
                 |
5. Configure upstream and proxy_pass
                 |
6. Validate (nginx -t)
                 |
7. Reload Nginx
                 |
8. Curl the LBR endpoint
```

| Error | Possible cause |
|---|---|
| Connection refused | No listener or active rejection |
| Connection timed out | Firewall, routing, or unresponsive backend |
| 502 Bad Gateway | Invalid/unreachable upstream response |
| 504 Gateway Timeout | Backend response took too long |
| 404 Not Found | Requested resource or route not found |

**Lab lesson:** After a lab restart, IP addresses and ports can change. Always rediscover them. Never accidentally configure the LBR's own IP as a backend.

## 6. Production concepts

A production load balancer needs more than basic request distribution:

- **Health checks:** Detect failed backends. Nginx Open Source supports passive failure detection; active checks require additional capabilities.
- **Timeouts:** Configure connection and response limits.
- **TLS termination:** Handle HTTPS at Nginx and optionally encrypt traffic to backends.
- **Session management:** Prefer shared session storage (e.g. Redis) or stateless authentication over server-local sessions.
- **High availability:** Use redundant load balancers and failover mechanisms.
- **Monitoring:** Track request rate, errors, latency, and server resource usage.

Example timeout configuration:

```nginx
location / {
    proxy_pass http://app_servers;
    proxy_connect_timeout 5s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

## 7. DevOps and MLOps relevance

The same architecture applies to ML model-serving APIs:

```text
       Applications / Users
                |
          Nginx / Gateway
                |
        +-------+-------+
        |       |       |
      Model   Model   Model
      API 1   API 2   API 3
        |       |       |
      Model   Model   Model
```

**Why it matters:**
- Scale inference by running multiple model replicas.
- Distribute prediction requests.
- Keep a stable API endpoint.
- Support rolling, blue-green, canary, and shadow deployments.
- Monitor inference latency, errors, throughput, CPU, and GPU utilization.

### Related tools

| Tool | Purpose |
|---|---|
| HAProxy | Load balancing |
| Keepalived | High availability / virtual IP failover |
| Kubernetes Service | Stable endpoint for pods |
| Ingress Controller | HTTP/HTTPS routing into Kubernetes |
| Prometheus + Grafana | Monitoring and dashboards |
| Redis | Shared sessions and caching |
| Ansible / Terraform | Configuration and infrastructure automation |

## Quick revision

Remember these five things:

1. `upstream` defines **where requests can go**.
2. `proxy_pass` defines **where requests are forwarded**.
3. `nginx -t` validates configuration.
4. `systemctl reload nginx` applies changes.
5. Test backend connectivity before troubleshooting the load balancer.

**Takeaway:** Nginx provides a stable entry point and distributes traffic across application instances. This is a foundational pattern for scalable DevOps infrastructure and production ML inference systems.