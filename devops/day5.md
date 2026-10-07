# Day 5 — SELinux Configuration

## 🎯 Task

Install SELinux packages and **permanently disable SELinux** without rebooting.

## 🧠 Concepts

- SELinux
- `yum`
- SELinux configuration
- Permanent vs temporary changes
- `/etc/selinux/config`

## 🔧 Commands

### Install SELinux packages
```bash
sudo yum install -y selinux-policy selinux-policy-targeted
```

### Disable SELinux permanently
```bash
sudo vi /etc/selinux/config
```

Set:
```text
SELINUX=disabled
```

### Verify configuration
```bash
grep "^SELINUX=" /etc/selinux/config
```

Expected:
```text
SELINUX=disabled
```

## ⚠️ Important

Changing `/etc/selinux/config` affects the **next boot**.

So `getenforce` may still show `Enforcing` until the server is rebooted.

The challenge specifically says **do not reboot**.

## 📝 Key Takeaway

- `/etc/selinux/config` → controls SELinux state after boot.
- `SELINUX=disabled` → permanently disables SELinux.
- A reboot is required for the change to take effect.
- If a challenge says no reboot, only change the configuration and leave the server running.

## 🔐 SELinux — Extra Notes

SELinux is a **Mandatory Access Control (MAC)** system that enforces security policies on Linux.

### Common commands

```bash
getenforce
ls -Z
semanage
restorecon
chcon
```

- `getenforce` → check SELinux mode
- `ls -Z` → view SELinux security contexts
- `semanage` → configure SELinux settings/contexts permanently
- `restorecon` → apply the correct context
- `chcon` → manually change a context

### 🧠 Important

You normally **use the existing SELinux policy** rather than writing your own.

If the existing policy doesn't support an application's requirements, advanced users can create **custom SELinux policies**.

For persistent context configuration, prefer:

```bash
semanage fcontext ...
restorecon ...
```

rather than relying on `chcon`.

### Mental Model

```text
Application
     ↓
SELinux Policy
     ↓
Security Context
     ↓
ALLOW / DENY
```

**SELinux = an extra security layer that controls what processes are allowed to access or do.**