# DevOps Day 21 — Create a Bare Git Repository

## 🎯 Task

Install Git on the Storage Server and create a bare repository named `/opt/apps.git`.

## 🧠 Concepts

- Git repositories
- Bare vs non-bare repositories
- Central (remote) repositories
- `yum`
- Git internals (`HEAD`, `objects/`, `refs/`)

## 🔧 Commands

### Install Git

```bash
sudo yum install -y git
git --version
```

### Create the bare repository

```bash
sudo git init --bare /opt/apps.git
```

### Verify

```bash
ls -la /opt/apps.git
git -C /opt/apps.git rev-parse --is-bare-repository
```

Expected:

```text
true
```

## 📦 Bare vs Non-Bare

| Type | Working directory | Typical use |
|---|---|---|
| Non-bare (`git init`) | Yes, files are checked out beside `.git/` | Local development |
| Bare (`git init --bare`) | No, only Git data | Central repo that developers push to and pull from |

A bare repo has no checked-out files, so pushes never conflict with a working tree.

## ⚠️ Common Issues

- **Wrong path or name:** the repo must be exactly `/opt/apps.git`.
- **Used `git init` without `--bare`:** you get `/opt/apps.git/.git` instead of a bare repo. Delete it and recreate.
- **Permission denied under `/opt`:** use `sudo`.
- **`fatal: detected dubious ownership`:** the repo is owned by `root` but Git is run as another user. Git blocks this by design. Run the command with `sudo`, or add an exception with `git config --global --add safe.directory /opt/apps.git`. The repo itself is fine.
- **Repo owned by root:** fine for this task, but other users need `chown` or group permissions to push.

## 📝 Key Takeaway

**A bare repository is the server-side, shared copy of a project: no working files, only Git history, meant to be the push/pull target for developers.**