# DevOps Day 13 — iptables Firewall Configuration

## Objective
Secure Apache on port `3000` across all three app servers (`stapp01`, `stapp02`, `stapp03`):
- Allow incoming traffic only from the Load Balancer (LBR).
- Block all other sources.
- Preserve firewall rules across reboots.

## 1. iptables fundamentals

`iptables` is a Linux firewall tool that controls network traffic using rules.

**Important concepts:**

| Concept | Meaning |
|---|---|
| INPUT | Traffic entering the local server |
| OUTPUT | Traffic leaving the local server |
| FORWARD | Traffic routed through the server |
| ACCEPT | Allow matching traffic |
| DROP | Silently discard matching traffic |
| REJECT | Deny traffic and send a rejection response |
| `-p tcp` | Match TCP traffic |
| `-s` | Match source IP address |
| `--dport` | Match destination port |
| `-I` | Insert a rule at a specified position (default: first) |
| `-A` | Append a rule to the end of a chain |
| `-L` | List firewall rules |
| `-n` | Display numeric IP addresses and ports |

## 2. Install firewall packages

On each app server:

```bash
sudo yum install -y iptables iptables-services
```

- `iptables`: manages firewall rules.
- `iptables-services`: provides systemd integration and persistent rule loading.

## 3. Write firewall rules

LBR IP: `10.244.195.12`

```bash
sudo iptables -I INPUT -p tcp -s 10.244.195.12 --dport 3000 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3000 -j DROP
```

**How to read a rule:**

```bash
iptables -I INPUT -p tcp -s SOURCE_IP --dport PORT -j ACTION
```

- `INPUT`: inspect incoming traffic.
- `-p tcp`: match TCP.
- `-s SOURCE_IP`: match the sender.
- `--dport PORT`: match the destination port.
- `-j ACTION`: apply ACCEPT, DROP, or REJECT.

**Why rule order matters:**

iptables evaluates rules from top to bottom and applies the first matching terminating action.

1. Insert the LBR ACCEPT rule first.
2. Append the general DROP rule afterward.

This ensures the LBR is allowed while everyone else is blocked on port 3000.

## 4. Verify active rules

```bash
sudo iptables -L INPUT -n --line-numbers
```

Expected logic:

```text
1 ACCEPT tcp -- 10.244.195.12  0.0.0.0/0 tcp dpt:3000
2 DROP   tcp -- 0.0.0.0/0     0.0.0.0/0 tcp dpt:3000
```

The default INPUT policy remains ACCEPT, so unrelated traffic is not blocked by this configuration.

## 5. Make rules persistent

Runtime firewall rules can disappear after a reboot. Save them and enable the service:

```bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
sudo systemctl enable iptables
sudo systemctl start iptables
```

Verify persistence and service status:

```bash
sudo systemctl is-enabled iptables
sudo systemctl status iptables
```

- `iptables-save`: exports the active rules.
- `/etc/sysconfig/iptables`: stores the saved rules on these hosts.
- `systemctl enable`: configures the service to start at boot.
- `systemctl start`: starts the service and applies saved rules.

`Active: active (exited)` is normal for this oneshot-style service: it applies the rules and exits successfully.

## 6. Useful troubleshooting commands

```bash
sudo iptables -L -n -v --line-numbers
sudo iptables-save
sudo systemctl status iptables
sudo journalctl -u iptables
```

| Problem | What to check |
|---|---|
| Traffic unexpectedly blocked | Rule order and source IP |
| Rules disappear after reboot | Saved rules file and enabled service |
| Firewall service fails | `systemctl status` and journal logs |
| Allowed host cannot connect | Confirm its actual source IP and TCP port |

## DevOps concepts to remember

- **Least privilege:** allow only the required source to access a service.
- **Default policy vs explicit rules:** a chain can have a permissive default policy while specific ports are restricted.
- **Rule ordering:** an earlier matching rule can determine the outcome before later rules are evaluated.
- **Runtime vs persistent configuration:** active rules and saved rules are separate; both must be handled.
- **Idempotency:** repeatedly appending rules can create duplicates. Inspect existing rules before applying them again.
- **Network troubleshooting:** distinguish application/listener issues, firewall filtering, and routing problems.

**Challenge result:** All three app servers have iptables installed, LBR-only access to TCP port 3000, and persistent firewall rules enabled.