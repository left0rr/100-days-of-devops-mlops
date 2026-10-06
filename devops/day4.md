# Day 4 — Grant Execute Permissions to a Script

## 🎯 Task

Grant executable permissions to `/tmp/xfusioncorp.sh` on **App Server 2** in the Stratos Datacenter.

The script must be executable by **all users**.

## 🧠 Concepts

- Linux file permissions
- `chmod`
- Read / Write / Execute permissions
- Owner / Group / Others
- Numeric permissions
- `755` permissions
- `sudo`
- Executable scripts
- Checking permissions with `ls -l`
- Checking numeric permissions with `stat`

## 🔧 Commands

### SSH into App Server 2

```bash
ssh steve@stapp02
```

### Check the current permissions

```bash
ls -l /tmp/xfusioncorp.sh
```

Example:

```text
---------- 1 root root 40 Oct 6 10:41 /tmp/xfusioncorp.sh
```

The first section represents the file permissions.

### Grant executable permissions

Because the file belongs to `root`, use `sudo`:

```bash
sudo chmod 755 /tmp/xfusioncorp.sh
```

### Verify

```bash
ls -l /tmp/xfusioncorp.sh
```

Expected:

```text
-rwxr-xr-x 1 root root ... /tmp/xfusioncorp.sh
```

You can also check the numeric permission:

```bash
stat -c "%a %n" /tmp/xfusioncorp.sh
```

Expected:

```text
755 /tmp/xfusioncorp.sh
```

---

## 🔐 Understanding Linux Permissions

Linux permissions have three categories:

```text
USER        GROUP        OTHERS
 ↓            ↓             ↓
rwx          r-x           r-x
```

- **User (u)** = owner of the file
- **Group (g)** = users belonging to the file's group
- **Others (o)** = everyone else

For example:

```text
-rwxr-xr-x
```

Breaks down into:

```text
- rwx r-x r-x
  └─┘ └─┘ └─┘
   u    g    o
```

The first `-` means this is a regular file.

## 📖 Permission Letters

There are three basic permissions:

```text
r = read
w = write
x = execute
```

For a regular file:

- `r` → user can read the file
- `w` → user can modify the file
- `x` → user can execute the file

---

## 🔢 Understanding Permission Numbers

Linux can represent permissions using numbers.

Each permission has a value:

```text
r = 4
w = 2
x = 1
```

Add them together to create a permission number.

### Examples

```text
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2 + 0 = 6
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4
-wx = 0 + 2 + 1 = 3
-w- = 0 + 2 + 0 = 2
--x = 0 + 0 + 1 = 1
--- = 0 + 0 + 0 = 0
```

The three-digit number represents:

```text
USER   GROUP   OTHERS
 ↓       ↓       ↓
 7       5       5
```

So:

```text
755
```

means:

```text
7 = rwx
5 = r-x
5 = r-x
```

Therefore:

```text
755 = rwxr-xr-x
```

or:

```text
-rwxr-xr-x
```

---

## ⭐ Common Permission Numbers

### 755 — Executable files/scripts

```text
755 = rwxr-xr-x
```

Owner can:

```text
read + write + execute
```

Group can:

```text
read + execute
```

Others can:

```text
read + execute
```

Very common for:

- Bash scripts
- Executable programs
- Directories that users need to access

Command:

```bash
chmod 755 script.sh
```

### 644 — Normal files

```text
644 = rw-r--r--
```

Owner:

```text
read + write
```

Group:

```text
read
```

Others:

```text
read
```

Common for:

- Configuration files
- Text files
- HTML files

Command:

```bash
chmod 644 file.txt
```

### 700 — Private executable

```text
700 = rwx------
```

Only the owner can read, write, or execute.

```bash
chmod 700 script.sh
```

### 600 — Private file

```text
600 = rw-------
```

Only the owner can read/write.

Common for sensitive files such as private keys.

---

## 🛠️ chmod Symbolic vs Numeric

There are two common ways to use `chmod`.

### Numeric

```bash
chmod 755 script.sh
```

This explicitly sets the permissions to `755`.

### Symbolic

```bash
chmod u+x script.sh
```

Add execute permission for the owner.

```bash
chmod g+x script.sh
```

Add execute permission for the group.

```bash
chmod o+x script.sh
```

Add execute permission for others.

```bash
chmod a+x script.sh
```

Add execute permission for everyone.

Where:

```text
u = user/owner
g = group
o = others
a = all
```

---

## ⚠️ Important Lesson From This Challenge

Initially, the file looked like:

```text
----------
```

Using:

```bash
sudo chmod +x /tmp/xfusioncorp.sh
```

resulted in:

```text
---x--x--x
```

This technically gave everyone execute permission, but the KodeKloud checker expected the standard executable permission:

```text
-rwxr-xr-x
```

So the correct command for the challenge was:

```bash
sudo chmod 755 /tmp/xfusioncorp.sh
```

This is a good reminder:

> When a challenge specifies a particular permission requirement, check whether it expects a specific mode such as `755`, rather than only checking whether `x` exists.

---

## 🔍 Useful Commands to Remember

### See permissions

```bash
ls -l filename
```

### See numeric permissions

```bash
stat -c "%a %n" filename
```

### Change permissions numerically

```bash
chmod 755 filename
```

### Add execute permission

```bash
chmod +x filename
```

### Add execute permission for everyone

```bash
chmod a+x filename
```

### Remove execute permission

```bash
chmod a-x filename
```

### Change owner

```bash
sudo chown user filename
```

### Change group

```bash
sudo chgrp group filename
```

---

## 🧠 Quick Memory Trick

Remember:

```text
r = 4
w = 2
x = 1
```

Then build the number.

For example:

```text
rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
```

Therefore:

```text
755

7 → rwx
5 → r-x
5 → r-x
```

So:

```text
755 = rwxr-xr-x
```

Think:

> **7 = everything, 5 = read + execute, 5 = read + execute**

## 📝 Key Takeaway

Linux permissions control what the **owner, group, and other users** can do with a file.

The three main permissions are:

```text
r = 4
w = 2
x = 1
```

The most important permission modes to remember for DevOps are:

```text
755 = rwxr-xr-x  → executable files/scripts
644 = rw-r--r--  → normal readable files
700 = rwx------  → private executable
600 = rw-------  → private file
```

For executable scripts, `755` is a common and useful default:

```bash
sudo chmod 755 /tmp/xfusioncorp.sh
```