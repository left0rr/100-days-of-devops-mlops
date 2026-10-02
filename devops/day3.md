# Day 3 — Disable Direct Root SSH Login

## 🎯 Task

Disable direct SSH login for the root user on all App Servers in the Stratos Datacenter.

## 🧠 Concepts

- SSH security
- Root login
- SSH daemon configuration
- sshd_config
- PermitRootLogin
- Restarting SSH services
- Security hardening

## 🔧 Commands

# SSH into the app server
ssh <username>@stapp01

# Edit SSH configuration
sudo vi /etc/ssh/sshd_config

# Set:
PermitRootLogin no

# Restart SSH service
sudo systemctl restart sshd

# Verify configuration
sudo sshd -T | grep permitrootlogin

# Expected output:
permitrootlogin no

# Repeat on all app servers
stapp01
stapp02
stapp03

## 📝 Key Takeaway

PermitRootLogin no prevents users from logging directly into the server as root through SSH, improving server security.
