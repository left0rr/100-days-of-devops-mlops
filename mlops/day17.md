# MLOps Day 17 — DVC Experiments

## Objective
Run multiple isolated experiments with different hyperparameter values, compare their evaluation metrics, and apply the best-performing experiment to the workspace.

## 1. Run experiments

From the project directory:

```bash
cd /root/code/fraud-detection
```

Create experiments with different `max_depth` values:

```bash
dvc exp run --name exp-depth-2 --set-param params.yaml:max_depth=2
dvc exp run --name exp-depth-6 --set-param params.yaml:max_depth=6
dvc exp run --name exp-depth-12 --set-param params.yaml:max_depth=12
```

**Key concepts:**
- `dvc exp run` executes the pipeline as an experiment.
- `--name` assigns a recognizable experiment name.
- `--set-param` overrides a parameter for that experiment without permanently editing `params.yaml`.
- Each experiment runs the pipeline and records its parameters, metrics, and outputs.

## 2. Compare experiment results

```bash
dvc exp show -A
```

Compare the recorded metrics, especially `f1_score`, to identify the best experiment.

| Experiment | `max_depth` | Accuracy | F1 score |
|---|---:|---:|---:|
| Baseline | 4 | 0.71 | 0.5735 |
| exp-depth-2 | 2 | 0.60 | 0.2453 |
| exp-depth-6 | 6 | 0.745 | 0.6483 |
| exp-depth-12 | 12 | 0.75 | 0.6795 |

**Result:** `exp-depth-12` performed best, with an F1 score of `0.6795`.

## 3. Apply the best experiment

```bash
dvc exp apply exp-depth-12
```

This restores the selected experiment's results into the workspace, including its parameter values, metrics, and model outputs.

## 4. Verify the applied experiment

```bash
cat params.yaml
dvc metrics show
dvc status
```

Expected values:

- `max_depth: 12`
- `n_estimators: 100`
- `accuracy: 0.75`
- `f1_score: 0.6795`

## Important distinctions

- **Run:** creates an experiment from the current pipeline and parameter overrides.
- **Show:** compares experiment parameters and recorded metrics.
- **Apply:** brings a selected experiment's results into the working directory.
- **Baseline:** provides the reference result against which experiments can be evaluated.

**Takeaway:** DVC Experiments make hyperparameter exploration reproducible and comparable without manually creating separate project copies or permanently changing parameters for every trial.