# MLOps Day 10 — DVC: Initialization & Data Versioning

## 1. What DVC does

**DVC (Data Version Control)** works alongside Git to version large datasets and ML models.

- **Git:** Tracks code, configuration, and DVC metadata.
- **DVC:** Tracks data/model versions and stores artifact contents in a local cache.
- **DVC remote:** Optional external storage for sharing artifacts between machines.

## 2. Install and initialize

A Git repository should normally exist first. A virtual environment is recommended but optional.

```bash
cd /root/code/fraud-detection
python3 -m venv venv
source venv/bin/activate
python -m pip install dvc  # Install DVC in the active environment
dvc --version             # Verify installation

dvc init                  # Initialize DVC in the Git repository
git add .dvc .dvcignore   # Stage DVC initialization files
git commit -m "Initialize DVC"
```

`venv` isolates Python packages; it does not install DVC itself.

## 3. Why are there two `.gitignore` files?

**Root `.gitignore`:** `dvc add` typically adds the original dataset path here so Git doesn't track the large data file directly.

```bash
dvc add data/dataset.csv
git add data/dataset.csv.dvc .gitignore
git commit -m "Track dataset with DVC"
```

- `data/dataset.csv.dvc` — small metadata file tracked by Git.
- `.gitignore` — prevents Git from tracking the original dataset.
- DVC cache — stores the actual data contents.

**Why add `.gitignore` again?** Because `dvc add` may modify it. Committing that change ensures other clones also ignore the raw dataset.

**`.dvc/.gitignore`:** Protects internal DVC files from being committed. It is separate from the root `.gitignore`.

```bash
find .dvc -type f       # List files inside DVC's control directory
cat .gitignore          # Inspect root Git ignore rules
cat .dvc/.gitignore     # Inspect DVC's internal ignore rules
```

## 4. Check ignored files and data status

```bash
git status
git check-ignore -v data/dataset.csv  # Show which Git rule ignores the file
dvc status                            # Check tracked data/pipeline state
dvc diff                              # Compare tracked data between revisions
```

`git check-ignore -v` is useful when a file is unexpectedly ignored or appears in Git status.

## 5. Add, remove, and restore data

```bash
dvc add data/dataset.csv              # Start tracking data with DVC
git add data/dataset.csv.dvc .gitignore
git commit -m "Track dataset with DVC"
```

Remove DVC tracking:

```bash
dvc remove data/dataset.csv.dvc
```

Removes the DVC tracking file and related tracking configuration; it does not directly mean “delete this data from the cache.”

Remove tracking **and** the workspace output:

```bash
dvc remove --outs data/dataset.csv.dvc
```

Restore tracked outputs from the available cache:

```bash
dvc checkout
```

## 6. Cache and garbage collection

DVC normally stores cached artifacts under `.dvc/cache`.

```bash
du -sh .dvc/cache  # Check cache size
dvc status         # Check for changes
dvc gc -w          # Remove cache objects unused by the current workspace
```

**Warning:** `dvc gc -w` can permanently remove cached data needed by other branches, tags, or workspaces. Check `dvc gc --help` and select an appropriate scope before cleanup.

## 7. Remote storage

```bash
dvc push  # Upload cached artifacts to configured remote storage
dvc pull  # Download artifacts referenced by the current project version
```

These commands require a configured remote for sharing data between machines.

## Key takeaway

Git tracks code and DVC metadata; DVC manages data and model artifacts. The root `.gitignore` excludes raw datasets from Git, while `.dvc/.gitignore` protects internal DVC files. Use a virtual environment to isolate DVC's installation, and be careful with cache cleanup.