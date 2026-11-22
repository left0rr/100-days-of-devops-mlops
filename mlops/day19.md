# MLOps Day 19 — Production DVC Pipeline with SeaweedFS

## 1. Pipeline stages and dependencies

A DVC pipeline defines reproducible stages in `dvc.yaml`. Each stage specifies its command, dependencies, parameters, and outputs.

```yaml
stages:
  train:
    cmd: python3 scripts/train.py
    deps:
      - data/processed/clean.csv
      - scripts/train.py
    params:
      - n_estimators
      - max_depth
      - test_size
      - random_seed
    outs:
      - data/processed/test_split.csv
      - models/model.pkl
    metrics:
      - metrics.json:
          cache: false

  evaluate:
    cmd: python3 scripts/evaluate.py
    deps:
      - data/processed/test_split.csv
      - models/model.pkl
      - scripts/evaluate.py
    outs:
      - reports/evaluation.json:
          cache: false
```

**Key concepts:**
- `deps`: inputs that trigger a stage rerun when changed.
- `params`: configuration values tracked from `params.yaml`.
- `outs`: generated artifacts, cached by DVC by default.
- `metrics`: files containing measurable model performance results.
- DVC builds the execution DAG from dependencies and outputs.

When parameters are root-level keys in `params.yaml`, list them individually under `params`.

## 2. Add stages using the CLI

Example:

```bash
dvc stage add -n train \
  -p n_estimators,max_depth,test_size,random_seed \
  -d data/processed/clean.csv \
  -d scripts/train.py \
  -o models/model.pkl \
  -o data/processed/test_split.csv \
  -- python3 scripts/train.py
```

For evaluation:

```bash
dvc stage add -n evaluate \
  -d models/model.pkl \
  -d data/processed/test_split.csv \
  -d scripts/evaluate.py \
  -o reports/evaluation.json \
  -- python3 scripts/evaluate.py
```

**Important:** CLI `-o` creates cached outputs by default. Edit `dvc.yaml` when an output requires `cache: false`.

## 3. Metrics versus ordinary outputs

```yaml
metrics:
  - metrics.json:
      cache: false
```

- Metrics are intended for model evaluation and comparison.
- `cache: false` prevents DVC from storing the file in its cache.
- An output declared with `cache: false` is also not uploaded by `dvc push`.
- These files should be preserved through Git tracking.

Use cached outputs for large reproducible artifacts such as models and datasets; use uncached outputs for reports and metrics that should be tracked directly in Git.

## 4. Script placement and reproducibility

When challenge scripts are supplied in a staging directory:

```bash
mkdir -p scripts
cp scripts-staging/train.py scripts/train.py
cp scripts-staging/evaluate.py scripts/evaluate.py
```

**Use `cp`, not `mv`, when the original staging files are expected to remain present.** Some lab validators check the original starting files.

## 5. Reproduce and inspect the pipeline

```bash
dvc repro
dvc status
dvc dag
dvc metrics show
```

- `dvc repro`: executes changed stages in dependency order and skips unchanged stages.
- `dvc status`: checks whether data or pipeline stages are out of date.
- `dvc dag`: displays the dependency graph.
- `dvc metrics show`: displays tracked metrics.

A successful reproduction updates `dvc.lock`, recording the dependency, parameter, and output state used by the pipeline.

## 6. Push artifacts to remote storage

```bash
dvc remote list
dvc push
```

DVC pushes cached artifacts to the configured remote. In this challenge, the remote is SeaweedFS using an S3-compatible endpoint.

**Verification:** Open the SeaweedFS Filer and navigate to:

```text
/buckets/dvc-storage/
```

Look for uploaded objects under:

```text
files/md5/...
```

Remember: uncached outputs (`cache: false`) are not uploaded by `dvc push`.

## 7. Git release and version tagging

After successful reproduction and remote push:

```bash
git status
git add -A
git commit -m "Complete production DVC pipeline"
git tag v1.0
```

Verify:

```bash
git tag
git show --stat --oneline v1.0
git status
```

Commit the pipeline definition, lock file, parameters, scripts, and Git-tracked metrics/reports. DVC-managed cached artifacts are stored remotely rather than committed as ordinary Git files.

---

## MLOps engineer takeaways

- Build pipelines as dependency graphs, not isolated scripts.
- Track only the parameters that influence each stage.
- Separate model/data artifacts from reports and metrics.
- Use `dvc.lock` to preserve reproducible pipeline state.
- Push cached artifacts to remote storage for collaboration and recovery.
- Use Git commits and tags to identify reproducible release states.
- Preserve challenge-provided staging files when lab validation expects them.

**Production workflow:** Define stages → reproduce → inspect metrics → push artifacts → commit → tag → verify remote storage.