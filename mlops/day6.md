# MLOps Day 6 — Ruff & Black

## 🎯 Task

Configure and fix Python code so it passes **Ruff** and **Black** checks.

## 🧠 Concepts

- **Ruff** → Python linter; finds code-quality issues, unused imports, import problems, etc.
- **Black** → Python formatter; automatically formats code consistently.
- `pyproject.toml` → configuration file for both tools.

## ⚙️ Configuration

In `pyproject.toml`:

```toml
[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "W", "I"]

[tool.black]
line-length = 120
```

### Ruff rules

```text
E → style/errors
F → code errors such as unused imports
W → warnings
I → import sorting
```

## 🔧 Useful Commands

Check Ruff:

```bash
ruff check src/
```

Automatically fix Ruff issues:

```bash
ruff check src/ --fix
```

Format code with Black:

```bash
black src/
```

Check whether Black formatting is correct:

```bash
black --check src/
```

## 🔄 Typical Workflow

```text
Edit pyproject.toml
       ↓
ruff check src/ --fix
       ↓
black src/
       ↓
ruff check src/
       ↓
black --check src/
```

## 📝 Key Takeaway

**Ruff finds and fixes many Python code-quality problems.**

**Black formats Python code consistently.**

Use `pyproject.toml` to configure both, and always run the check commands before considering the code ready.