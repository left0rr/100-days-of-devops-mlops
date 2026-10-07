# Day 4 — Grant Execute Permissions to a Script

## 🎯 Task

Grant executable permissions to a script so **all users** can execute it.

## 🧠 Concepts

- Linux file permissions
- `chmod`
- Owner / Group / Others
- Numeric permissions
- `sudo`
- Checking permissions with `ls -l`

## 🔧 Commands

### Check permissions
```bash
ls -l <file>
```

### Grant standard executable permissions
```bash
sudo chmod 755 <file>
```

### Verify
```bash
ls -l <file>
stat -c "%a %n" <file>
```

Expected:
```text
755
```

## 🔐 Linux Permissions

```text
r = read    = 4
w = write   = 2
x = execute = 1
```

Permissions are grouped as:

```text
USER   GROUP   OTHERS
 ↓       ↓       ↓
rwx     r-x     r-x
```

### Common modes

```text
755 = rwxr-xr-x → executable files
644 = rw-r--r-- → normal files
700 = rwx------ → private executable
600 = rw------- → private file
```

## 🛠️ chmod

Numeric:
```bash
chmod 755 <file>
```

Symbolic:
```bash
chmod a+x <file>
```

`a+x` adds execute permission for everyone, while `755` sets the complete permission mode.

## ⚠️ Challenge Lesson

`chmod +x` may only add execute permission and can produce a mode like:

```text
---x--x--x
```

If the challenge expects the standard executable mode, use:

```bash
sudo chmod 755 <file>
```

## 📝 Key Takeaway

Remember:

```text
r = 4
w = 2
x = 1

755 = rwxr-xr-x
```

`755` is a common permission mode for executable scripts.