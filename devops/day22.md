# DevOps Day 22 — Clone a Git Repository

## 🎯 Task

As user `natasha` on the Storage Server, clone `/opt/blog.git` into `/usr/src/kodekloudrepos` without changing the repository, permissions, or existing directories.

## 🧠 Concepts

- `git clone`
- Local (filesystem-path) remotes
- Bare repo as the source, working copy as the clone
- Clone target naming (`blog/` subfolder vs `.`)
- Running as the right user
- Not modifying permissions or the source repo

## 🔧 Commands

```bash
ssh natasha@ststor01

cd /usr/src/kodekloudrepos
git clone /opt/blog.git
```

- `/opt/blog.git` is the source, a bare repository.
- Without a target argument, Git creates `/usr/src/kodekloudrepos/blog/`.

### Verify

```bash
ls -la /usr/src/kodekloudrepos
git -C /usr/src/kodekloudrepos/blog remote -v
git -C /usr/src/kodekloudrepos/blog status
```

Expected:

```text
origin  /opt/blog.git (fetch)
origin  /opt/blog.git (push)
On branch master
No commits yet
```

## 📦 Clone vs Bare

| Type | Purpose |
|---|---|
| `/opt/blog.git` (bare) | Central repo, history only, no working files |
| `/usr/src/kodekloudrepos/blog` (clone) | Working copy where developers edit and commit |

## ⚠️ Common Issues

- **Check failed after `git clone /opt/blog.git .`:** the checker expects `/usr/src/kodekloudrepos/blog`. Cloning into `.` put the repo directly in `kodekloudrepos`. Remove the mistaken `.git` (you created it) and clone again without the dot.
- **`warning: You appear to have cloned an empty repository`:** expected, the source repo has no commits yet.
- **Typo in the path:** `kidekloudrepos` does not exist, `kodekloudrepos` does. Use `ls /usr/src` to confirm.
- **Permission denied:** make sure you are `natasha`, not `thor` on the jump host. Do not `chmod` or `chown` to work around it.
- **`dubious ownership`:** fix with `git config --global --add safe.directory <path>`, which only edits your own user config.

## 📝 Key Takeaway

**Use the default `git clone <source>` form unless the task names an exact folder: the repo lands in a subfolder named after the source. Always verify the final path, not just that the clone succeeded.**