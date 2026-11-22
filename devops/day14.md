# DevOps Day 14 — Apache Service Troubleshooting

## Objective
Ensure Apache (`httpd`) is running on all three app servers and listening on TCP port `5001`.

## 1. Diagnose service status

```bash
sudo systemctl status httpd
sudo ss -lntp | grep httpd
```

- `systemctl status`: checks whether a service is active or failed.
- `ss -lntp`: displays listening TCP ports and associated processes.
- `grep`: filters the output for Apache.

**Finding:**
- `stapp02` and `stapp03`: Apache running on port `5001`.
- `stapp01`: Apache failed with `(98) Address already in use`.

## 2. Identify the port conflict

```bash
sudo ss -lntp | grep ':5001'
sudo lsof -i :5001
```

Found:

```text
127.0.0.1:5001  users:(("sendmail",pid=38487,...))
```

**Root cause:** Sendmail was listening on localhost port `5001`, preventing Apache from binding to the required port.

## 3. Resolve the conflict

Stop the conflicting service:

```bash
sudo systemctl stop sendmail
```

Start Apache and enable it at boot:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

- `stop`: releases the port occupied by Sendmail.
- `start`: starts Apache immediately.
- `enable`: configures Apache to start automatically after reboot.

## 4. Verify the solution

```bash
sudo systemctl status httpd
sudo ss -lntp | grep httpd
```

Expected:
- Apache reports `active (running)`.
- Apache listens on `*:5001`.
- No other process conflicts with the required port.

## Key concepts for DevOps

| Concept | Explanation |
|---|---|
| Port conflict | Two processes cannot normally bind the same IP address and port combination. |
| `ss` | Inspects sockets, listening ports, and associated processes. |
| `lsof` | Identifies processes using files, ports, and sockets. |
| `systemctl stop` | Stops a running service. |
| `systemctl start` | Starts a service immediately. |
| `systemctl enable` | Enables automatic startup at boot. |
| `localhost` | `127.0.0.1`, the server's loopback interface. |
| `*:5001` | Listening on port 5001 across all available IPv4 interfaces. |

## Troubleshooting lessons

- Read service logs carefully: `Address already in use` indicates a port-binding conflict.
- Identify the process occupying a port before stopping or killing it.
- A successful service configuration does not guarantee the service can start; runtime resources such as ports must also be available.
- Verify both service status and listening port, not just whether the service command succeeds.
- Avoid changing application configuration when the actual problem is another process occupying the required port.

**Challenge result:** Apache is configured to run on port `5001` across all three app servers, with automatic startup enabled.