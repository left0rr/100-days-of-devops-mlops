# DevOps Day 12 — Troubleshoot Apache Connectivity on Port 5000

## Goal
Restore Apache HTTP access on `stapp01:5000` without modifying the existing `index.html`.

## 1. Debugging Chain of Thought

**Follow the request from application to network:**

1. **Check the service:** `httpd` was `failed`.
2. **Read the logs:** `Address already in use` on port `5000` indicated a port conflict.
3. **Identify the listener:** `ss -lntp` showed `sendmail` bound to `127.0.0.1:5000`.
4. **Resolve the port conflict:** Stop Sendmail, then start Apache.
5. **Test locally:** Apache returned HTML from `127.0.0.1:5000`.
6. **Test remotely:** `curl` from the jump host still returned `No route to host`.
7. **Inspect firewall rules:** `iptables` showed a final `REJECT` rule, with no prior allow rule for port `5000`.
8. **Fix and verify:** Insert an `ACCEPT` rule before `REJECT`; retest from the jump host.

**Key lesson:** A running service and a listening socket do not guarantee remote connectivity. Diagnose each layer separately.

## 2. Commands Used

### Service and configuration

```bash
sudo systemctl status httpd --no-pager
sudo journalctl -u httpd -n 30 --no-pager
sudo grep -RniE '^[[:space:]]*Listen' /etc/httpd/conf /etc/httpd/conf.d
sudo apachectl configtest
```

- `systemctl`: inspect service state.
- `journalctl`: identify startup failures.
- `grep`: find active `Listen` directives.
- `apachectl configtest`: validate Apache configuration.

### Find and resolve the port conflict

```bash
sudo ss -lntp | grep ':5000'
sudo ps -fp <PID>
sudo systemctl status sendmail --no-pager
sudo systemctl stop sendmail
sudo systemctl enable --now httpd
```

`ss -lntp` identifies listening TCP ports and owning processes. Replace `<PID>` with the actual process ID.

### Verify local and remote access

```bash
curl -v http://127.0.0.1:5000
sudo ss -lntp | grep ':5000'
```

From the jump host:

```bash
curl -v --connect-timeout 5 http://stapp01:5000
```

### Diagnose and fix the firewall

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

Insert the allow rule before the existing final `REJECT`:

```bash
sudo iptables -I INPUT 5 -p tcp --dport 5000 -j ACCEPT
```

Verify rule order:

```bash
sudo iptables -L INPUT -n -v --line-numbers
```

**Why rule order matters:** iptables processes rules from top to bottom. The traffic must match the port-5000 `ACCEPT` rule before reaching the catch-all `REJECT`.

## 3. Error-to-Cause Reference

| Symptom | Likely cause / next check |
|---|---|
| `httpd` failed | Check `systemctl status` and `journalctl` |
| `Address already in use` | Another process owns the listening port; inspect with `ss` |
| `Connection refused` | No accepting listener at the destination address/port, or an active rejection |
| `No route to host` | May indicate firewall rejection or a network/routing problem; inspect firewall rules and connectivity |
| Local curl works, remote curl fails | Investigate firewall rules, network filtering, and interface binding |
| `Syntax OK` but service fails | Configuration syntax is valid; check runtime errors such as port conflicts |

## 4. Important Concepts

- **Apache (`httpd`):** Web server serving HTTP requests.
- **Port binding:** Only compatible listeners can share the same address and port.
- **`127.0.0.1`:** Loopback address, reachable only from the same host.
- **`0.0.0.0` / `*`:** Listener accepts connections on available IPv4 interfaces / addresses, respectively.
- **iptables:** Packet-filtering rules; rule order and default/reject policies affect connectivity.
- **Local vs. remote tests:** Local `curl` checks the application path; remote `curl` also tests network access and firewall filtering.

## 5. Final Verification

- `httpd` is active and enabled.
- Apache listens on port `5000`.
- Local curl returns the existing webpage.
- The INPUT chain allows TCP port `5000` before the reject rule.
- `curl http://stapp01:5000` from the jump host returns the webpage.

**Outcome:** Apache connectivity restored without changing `index.html`.

**Operational note:** The iptables rule was added at runtime. Persist firewall rules only using the environment's supported mechanism if they must survive reboot. Sendmail was stopped to release the conflicting port; verify that disabling or stopping it is acceptable for the server's intended role before making that change permanent.