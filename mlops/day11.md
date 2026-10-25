# MLOps Day 11 — Track a Dataset with DVC

## Goal
Move `data/raw/transactions.csv` from Git tracking to DVC while keeping the dataset on disk.

## Steps

```bash
cd /root/code/fraud-detection
dvc add data/raw/transactions.csv
```

- `dvc add` creates `data/raw/transactions.csv.dvc` (the DVC pointer file) and updates `data/raw/.gitignore` to exclude the dataset from Git.
- The actual CSV remains in the working directory; DVC manages its content in the local cache.

If the CSV is still tracked by Git, explicitly remove it from Git's index **without deleting the local file**:

```bash
git rm --cached data/raw/transactions.csv
```

Stage and commit the DVC metadata and Git index change:

```bash
git add data/raw/transactions.csv.dvc data/raw/.gitignore
git commit -m "Track transactions dataset with DVC"
```

## Verify

```bash
git ls-files data/raw/transactions.csv      # Should return nothing
git ls-files data/raw/transactions.csv.dvc # Should show the pointer
git check-ignore -v data/raw/transactions.csv
dvc status
ls -lh data/raw/transactions.csv            # Confirm data remains on disk
git log -1 --oneline                        # Check commit message
```

## Key Takeaways

- **Git** tracks code, configuration, and the small `.dvc` pointer file.
- **DVC** manages the dataset content and its cache.
- `.gitignore` prevents the raw dataset from being added to Git again.
- `git rm --cached` removes a file from Git tracking but keeps the local copy.
- Commit both the `.dvc` pointer and `.gitignore` so collaborators can reproduce the tracking setup.