# MLOps Day 8 — Configure pre-commit Hooks

## Goal
Configure and register five pre-commit hooks to enforce code quality in the `fraud-detection` repository.

## 1. Configure `.pre-commit-config.yaml`

File location: `/root/code/fraud-detection/.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.17.0
    hooks:
      - id: ruff

  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 26.10.1
    hooks:
      - id: black
```

## 2. Install and run hooks

```bash
cd /root/code/fraud-detection

# Update repository revision pins
pre-commit autoupdate

# Register Git pre-commit hook
pre-commit install

# Run all configured hooks against tracked files
pre-commit run --all-files
```

Run `pre-commit run --all-files` again if a hook modifies files on the first run.

## 3. What each hook does

| Hook | Purpose |
|---|---|
| `trailing-whitespace` | Removes trailing spaces |
| `end-of-file-fixer` | Ensures files end with a newline |
| `check-yaml` | Validates YAML syntax |
| `ruff` | Lints Python code |
| `black` | Formats Python code consistently |

## Key takeaways
- Every repository entry must include a `rev:` release pin.
- Hook IDs must match the repository's supported IDs.
- `pre-commit autoupdate` refreshes release pins.
- `pre-commit install` registers the Git hook to run before commits.
- The first run may modify files; rerun the checks to verify the fixes.

**Result:** Configuration updated, Git hook installed, and all five hooks passed on the initial validation except `trailing-whitespace`, which fixed `process.py`. A final rerun is needed to confirm all five pass.
# MLOps Day 8 — Configure pre-commit Hooks

## Goal
Configure and register five pre-commit hooks to enforce code quality in the `fraud-detection` repository.

## 1. Configure `.pre-commit-config.yaml`

File location: `/root/code/fraud-detection/.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.17.0
    hooks:
      - id: ruff

  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 26.10.1
    hooks:
      - id: black
```

## 2. Install and run hooks

```bash
cd /root/code/fraud-detection

# Update repository revision pins
pre-commit autoupdate

# Register Git pre-commit hook
pre-commit install

# Run all configured hooks against tracked files
pre-commit run --all-files
```

Run `pre-commit run --all-files` again if a hook modifies files on the first run.

## 3. What each hook does

| Hook | Purpose |
|---|---|
| `trailing-whitespace` | Removes trailing spaces |
| `end-of-file-fixer` | Ensures files end with a newline |
| `check-yaml` | Validates YAML syntax |
| `ruff` | Lints Python code |
| `black` | Formats Python code consistently |

## Key takeaways
- Every repository entry must include a `rev:` release pin.
- Hook IDs must match the repository's supported IDs.
- `pre-commit autoupdate` refreshes release pins.
- `pre-commit install` registers the Git hook to run before commits.
- The first run may modify files; rerun the checks to verify the fixes.

**Result:** Configuration updated, Git hook installed, and all five hooks passed on the initial validation except `trailing-whitespace`, which fixed `process.py`. A final rerun is needed to confirm all five pass.
