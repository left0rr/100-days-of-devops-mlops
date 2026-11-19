# MLOps Day 18 — Versioning Datasets and Models with Git + DVC

## Objective

Maintain reproducible versions of datasets and trained models using Git branches and DVC:

- Tag the baseline as `v1.0`.
- Create a `v2-improved` branch.
- Replace the baseline dataset with improved data.
- Retrack the dataset and rerun the ML pipeline.
- Commit the new dataset/model metadata.
- Return to `main` and restore the original state.

## 1. Navigate and inspect the project

```bash
cd /root/code/fraud-detection

git status
git branch --show-current
git log -1 --oneline
dvc status
```

Confirm the working tree is clean and the DVC pipeline is up to date before tagging.

## 2. Preserve the baseline version

```bash
git tag v1.0
git tag
git show v1.0:dvc.lock
```

A Git tag marks a specific commit as a stable version. It preserves the original DVC metadata and pipeline state.

## 3. Create a new dataset/model branch

```bash
git switch -c v2-improved
```

Check branches:

```bash
git branch
```

Branches allow independent development and comparison without overwriting the baseline history.

## 4. Replace and retrack the dataset

Copy the improved dataset over the tracked dataset:

```bash
cp data/raw/transactions_v2.csv data/raw/transactions.csv
```

Track the replacement with DVC:

```bash
dvc add data/raw/transactions.csv
```

DVC calculates a new content hash and updates the `.dvc` metadata file.

**Important:** Keep the original staged `transactions_v2.csv` file. Do not delete it.

## 5. Retrain the model through the DVC pipeline

```bash
dvc repro
dvc status
```

`dvc repro` checks dependencies and outputs, then runs stages affected by changed inputs.

Typical pipeline:

1. `process_data` — clean/process the raw dataset.
2. `split_data` — generate training and test data.
3. `train` — train and generate `models/model.pkl`.

DVC updates `dvc.lock` with the new dependency and output hashes.

## 6. Commit the v2 version

```bash
git status
git add .
git commit -m "Add improved dataset and retrained model"
```

Verify:

```bash
git log --oneline -2
git status
```

Git commits the version metadata; DVC manages the actual large dataset and model contents.

## 7. Switch back and restore v1

```bash
git switch main
dvc checkout
dvc status
```

**Critical distinction:**

- `git switch` changes the Git branch and restores tracked metadata.
- `dvc checkout` restores DVC-managed files to match the current Git revision.
- `dvc status` checks whether the workspace matches the pipeline metadata.

## 8. Verify hashes and version history

Inspect the original dataset metadata:

```bash
git show v1.0:data/raw/transactions.csv.dvc
```

Compare with the actual dataset:

```bash
md5sum data/raw/transactions.csv
```

Inspect the original pipeline lockfile:

```bash
git show v1.0:dvc.lock
```

Inspect the v2 pipeline metadata:

```bash
git show v2-improved:dvc.lock
```

Matching hashes confirm that the expected file contents have been restored.

## Essential command reference

| Command | Purpose |
|---|---|
| `git status` | Check working-tree changes |
| `git tag v1.0` | Mark a baseline commit |
| `git switch -c branch` | Create and switch branches |
| `git switch main` | Return to baseline branch |
| `git show ref:file` | Inspect a file at a Git revision |
| `dvc add file` | Track data/model with DVC |
| `dvc repro` | Rebuild affected pipeline stages |
| `dvc checkout` | Restore DVC outputs for current Git revision |
| `dvc status` | Check data and pipeline consistency |
| `md5sum file` | Verify file content hash |
| `git log --oneline` | Inspect commit history |

## Key MLOps / DevOps concepts

**Git vs DVC**
- Git versions source code, configuration, pipeline definitions, and small metadata files.
- DVC versions large datasets and model artifacts using content hashes.
- `.dvc` files and `dvc.lock` connect Git revisions to specific data/model versions.

**Reproducibility**
A model version is meaningful only when its dataset, pipeline dependencies, parameters, and outputs can be traced together.

**Branch-based experimentation**
Separate branches allow teams to compare dataset and model versions, test improvements, and roll back without losing the baseline.

**DVC cache**
DVC stores content-addressed objects in its cache. `dvc checkout` uses those cached objects to restore files. `dvc push` and `dvc pull` synchronize them with remote storage.

**Production relevance**
This workflow supports:
- Dataset/model lineage.
- Reproducible training.
- Experiment isolation.
- Model rollback.
- Controlled promotion of improved models.
- Auditable ML releases.

**Operational rule:** Before switching versions, ensure important changes are committed. After switching branches, use `dvc checkout` to synchronize the workspace with that branch's DVC metadata.