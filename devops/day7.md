# Day 7 — Passwordless SSH Authentication

## 🎯 Task

Configure passwordless SSH from the jump host user `thor` to all app servers using their respective users.

## 🧠 Concepts

- SSH (Secure Shell)
- Public and private keys
- Key-based authentication
- `ssh-keygen`, `ssh-copy-id`
- SSH permissions
- Passwordless SSH vs passwordless `sudo`

## 🔧 Commands

### Generate an SSH key pair

```bash
ssh-keygen -t rsa
```

Accept the default location. Leave the passphrase empty if passwordless automation is required.

### Copy the public key to each server

```bash
ssh-copy-id user@server
```

Enter the remote user's password during initial setup.

### Connect and test

```bash
ssh user@server
```

Repeat for each app server using its correct username.

### Check existing keys

```bash
ls -l ~/.ssh/
```

## 🔑 SSH Key Types

- **RSA** — widely supported; `ssh-keygen -t rsa`
- **Ed25519** — modern, fast, and generally recommended; `ssh-keygen -t ed25519`
- **ECDSA** — another elliptic-curve key type

The public key (`.pub`) is copied to remote servers. The private key must remain secret on the jump host.

## 🔐 Important SSH Permissions

Typical permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

Never share or copy your private key to remote servers.

## ⚠️ Common Issues

- **Password still requested:** verify the public key is installed for the correct remote user.
- **Permission denied:** check usernames, key installation, and SSH file permissions.
- **Wrong key:** inspect `~/.ssh/` before generating another key to avoid overwriting an existing one.
- **`sudo` still asks for a password:** SSH authentication and sudo authorization are separate.

## 📝 Key Takeaway

**Passwordless SSH uses a private/public key pair to authenticate without entering the remote account password each time.** It is useful for automation, scripts, and server administration.
