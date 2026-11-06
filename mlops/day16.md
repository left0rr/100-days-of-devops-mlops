# MLOps Day 16 — DVC Metrics Tracking

## Objective
Configure DVC to recognize `metrics.json` as a metrics output of the `train` stage, rather than a regular file output.

No Python changes required.

## 1. Register metrics in `dvc.yaml`

Inside the existing `train` stage, add:

```yaml
metrics:
  - metrics.json:
      cache: false
```

Keep the existing `cmd`, `deps`, `params`, and model `outs` unchanged.

**Why this matters:**
- `metrics:` tells DVC that the file contains model evaluation results.
- `cache: false` prevents DVC from storing the metrics file in its cache.
- The JSON remains a regular Git-tracked file, making metric history easy to review.

## 2. Reproduce the pipeline

```bash
dvc repro
```

DVC executes the pipeline as needed, regenerates the metrics, and updates `dvc.lock`.

## 3. Display and compare metrics

```bash
dvc metrics show
dvc metrics diff
```

Example result:

```text
Path          accuracy    f1_score
metrics.json  1.0         1.0
```

- `dvc metrics show`: displays the current metric values.
- `dvc metrics diff`: compares workspace metrics against the latest Git commit (`HEAD`).

A `-` under `HEAD` means no committed baseline exists yet.

## Key concepts

| Concept | Purpose |
|---|---|
| `outs:` | Declares regular pipeline outputs |
| `metrics:` | Declares model evaluation outputs |
| `cache: false` | Keeps metrics outside DVC cache |
| `dvc.lock` | Records pipeline execution state |
| `dvc metrics show` | Displays current model metrics |
| `dvc metrics diff` | Compares metrics across Git revisions |

## DevOps / MLOps takeaway

- Separate model artifacts from evaluation metrics.
- Track model files with DVC and keep small, readable metrics in Git.
- Use metric comparisons to evaluate whether a new model performs better or worse.
- Commit the metrics file and relevant pipeline metadata to establish a Git baseline for future comparisons.

**Challenge result:** DVC successfully recognizes `metrics.json` and reports `accuracy = 1.0` and `f1_score = 1.0`.