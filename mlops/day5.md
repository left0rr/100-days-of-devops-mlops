# MLOps Day 5 — Makefile Automation

## 🎯 Task

Fix a Makefile to automate the ML workflow:

```text
setup → data → train → test
```

## 🧠 Concepts

- Makefiles
- `make`
- Make targets
- `.PHONY`
- Dependencies between targets
- Virtual environments
- Automation
- Makefile recipes require **TAB indentation**

## 🔧 Commands

Run the workflow:

```bash
make all
```

Check the Makefile:

```bash
cat Makefile
```

Common targets:

```bash
make setup
make data
make train
make test
make clean
make all
```

## 📌 Target Roles

```text
setup → create venv + install dependencies
data  → process data
train → train model
test  → run tests
clean → remove generated/cache files
all   → run setup → data → train → test
```

Declare targets as phony:

```makefile
.PHONY: setup data train test clean all
```

## ⚠️ Important

Makefile commands must start with a **real TAB**, not spaces.

`.PHONY` prevents Make from confusing a target with a file of the same name.

## 📝 Key Takeaway

**Makefile = a simple automation system for running repetitive project tasks in the correct order.**