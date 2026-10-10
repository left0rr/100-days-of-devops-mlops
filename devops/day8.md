# DevOps Day 8 — Install Ansible Globally

**Goal:** Install Ansible `4.10.0` on the Jump host using `pip3`, accessible to all users.

### Commands
```bash
sudo pip3 install ansible==4.10.0

ansible --version
python3 -m pip show ansible
ls -l /usr/local/bin/ansible
```

### Verification
- Package version: `4.10.0`
- Ansible Core: `2.11.12`
- Python package location: `/usr/local/lib/python3.9/site-packages`
- Executable: `/usr/local/bin/ansible`
- Permissions: `755` — executable by all users
- Tested successfully as `thor` and `nobody`.

### Key takeaways
- `sudo pip3 install` installs the package system-wide.
- `/usr/local/bin` is a standard location for globally available commands.
- `755` permissions allow the owner to read/write/execute and everyone else to read/execute.
- A user's `PATH` and home-directory permissions can affect command execution even when a package is installed correctly.
- Ansible Core `2.11.12` is the dependency installed with Ansible `4.10.0`; these are different version numbers.

**Result:** Ansible 4.10.0 is installed and verified.
