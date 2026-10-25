# DevOps Day 10 — Archive Website Content with Bash

## Goal
Create `/scripts/ecommerce_archive.sh` on App Server 3 to ZIP `/var/www/html/ecommerce`, save it locally, and copy it to the Nautilus Storage Server without an interactive password.

## Script

```bash
#!/bin/bash
set -e

cd /var/www/html
zip -r /archives/xfusioncorp_ecommerce.zip ecommerce
scp /archives/xfusioncorp_ecommerce.zip natasha@ststor01:/archives/xfusioncorp_ecommerce.zip
```

- `#!/bin/bash` — shebang; specifies Bash as the interpreter.
- `set -e` — stops the script when a command fails in common cases.
- `cd /var/www/html` — makes the archive contain the `ecommerce/` directory and its contents.
- `zip -r` — recursively archives files and subdirectories.
- `scp` — copies the archive over SSH to the remote `/archives/` directory.

## Setup and Verification

```bash
sudo yum install -y zip                 # Install prerequisite manually
ssh-keygen -t ed25519                   # Generate key as banner
ssh-copy-id natasha@ststor01            # Configure passwordless SSH
sudo mkdir -p /scripts /archives        # Prepare directories
sudo chown banner:banner /scripts /archives
chmod 755 /scripts/ecommerce_archive.sh # Make executable
/scripts/ecommerce_archive.sh           # Run as banner, without sudo
```

Verify both copies:

```bash
ls -lh /archives/xfusioncorp_ecommerce.zip
ssh natasha@ststor01 'ls -lh /archives/xfusioncorp_ecommerce.zip'
```

Both copies were present and 545 bytes. SSH/SCP completed without prompting for a password.

## Bash Scripting Essentials

- **Variables:** `NAME="value"`; reference with `"$NAME"`. Do not put spaces around `=`.
- **Quoting:** Double-quote variables and paths to avoid problems with spaces and shell expansion.
- **Exit codes:** `0` means success; nonzero usually means failure. `set -e` helps stop after errors.
- **Conditionals:** `if [ -f "$FILE" ]; then ...; fi` checks for a regular file; `-d` checks for a directory.
- **Command chaining:** `&&` runs the next command on success; `||` runs it on failure; `;` separates commands regardless of success.
- **Loops:** `for file in /path/*; do ...; done` processes matching items.
- **Here-documents:** `cat > script.sh <<'EOF' ... EOF` creates a multiline script without an editor. Quoting `EOF` prevents variable expansion while writing.
- **Permissions:** `chmod 755 script.sh` makes it executable. The running user also needs permission to read inputs and write outputs.
- **Passwordless SSH:** Configure the key for the actual script user; verify with `ssh natasha@ststor01 'hostname'`.
- **Ownership matters:** A user may write to a directory but still be unable to overwrite an existing file owned by someone else.

## Key Takeaway
Keep package installation and permission setup outside the script when required. Use absolute paths, quote variables, verify remote transfers, and run the script as the designated non-root user.