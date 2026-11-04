# MLOps Day 15 — DVC Parameter Tracking

## Goal
Connect `params.yaml` parameters to the `train` stage so DVC knows when a parameter change requires retraining.

## 1. Declare the parameter in `dvc.yaml`

Edit the existing `train` stage directly and add:

```yaml
params:
  - n_estimators
```

Keep the existing `cmd`, `deps`, and `outs` unchanged. DVC reads `n_estimators` from `params.yaml` by default.

**Why:** Without declaring the parameter, DVC may not track it as a dependency of the training stage.

## 2. Reproduce the pipeline

```bash
dvc repro
```

- With `n_estimators: 100`, the training stage runs with that value.
- Change it to `200` in `params.yaml` and rerun `dvc repro`.
- Only `train` should rerun because the upstream data stages have not changed.

## 3. Verify parameter changes

```bash
dvc params diff
dvc status
```

- `dvc params diff` compares parameter values between Git revisions and the current workspace.
- Example: `n_estimators` changes from `100` to `200`.
- `dvc status` checks whether pipeline stages or dependencies are out of date.

Inspect `dvc.lock` to verify the recorded parameter value and confirm `models/model.pkl` was regenerated.

## Key concepts

- **`params.yaml`** — stores configurable values separately from Python code.
- **`params:` in `dvc.yaml`** — declares which parameters a stage depends on.
- **`dvc.lock`** — records resolved stage details, including tracked parameter values.
- **Selective reproduction** — DVC reruns only stages affected by changed dependencies or parameters.
- **`dvc params diff`** — makes parameter changes between revisions easy to review.

## Common mistake

`dvc stage add` requires a complete stage definition, including its command. For an existing stage, directly editing `dvc.yaml` is simpler and avoids accidentally rebuilding the stage incorrectly.

**Takeaway:** Declare the parameters a stage uses, and DVC can track configuration changes and retrain only when necessary.