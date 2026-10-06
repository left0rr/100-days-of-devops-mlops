# Day 4 — Git & .gitignore

## 🎯 Task

Create a `.gitignore` for a Python/ML project and stop Git from tracking artifacts that were already committed, while keeping them on disk.

## 🧠 Concepts

- `.gitignore`
- Git tracking vs files on disk
- Git index
- `git add`
- `git rm --cached`
- `git status`
- `git ls-files`
- `git check-ignore`

## 📝 .gitignore

Common Python/ML patterns:

```gitignore
__pycache__/
*.pyc
venv/
.ipynb_checkpoints/
*.pkl
.env
```

⚠️ Pay attention to exact spelling.  
`__pycache__/` has **two underscores** on each side.

## 🔧 Useful Commands

### Check repository status

```bash
git status
```

### See tracked files

```bash
git ls-files
```

### Apply a new `.gitignore` to files already tracked

```bash
git rm -r --cached .
git add .
```

`--cached` means:

> Remove from Git's index, but keep the files on disk.

### Check ignored files

```bash
git status --ignored
```

### Find which `.gitignore` rule matches a file

```bash
git check-ignore -v <file>
```

### Commit changes

```bash
git commit -m "Add gitignore and remove tracked artifacts"
```

### Verify after committing

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

## ⭐ Key Takeaways

### `.gitignore` ≠ untracking

`.gitignore` prevents **untracked** files from being added.

If files are **already tracked**, use:

```bash
git rm -r --cached .
git add .
```

### `--cached`

```text
Git index     → remove
Filesystem    → keep
```

### Useful workflow

```bash
# Create/update .gitignore
vi .gitignore

# Apply it to already-tracked files
git rm -r --cached .
git add .

# Review
git status
git ls-files
git status --ignored

# Commit
git commit -m "Update gitignore and clean tracked artifacts"
```

## 🏆 Remember

For Python/ML projects, commonly ignore:

```text
Python cache
Virtual environments
Notebook checkpoints
Generated model artifacts
Local environment/secrets
```

**Most important lesson:**

> `.gitignore` prevents files from being tracked; `git rm --cached` stops Git tracking files that were already committed.