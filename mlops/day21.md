# MLOps Day 21 — Log a Run to MLflow

## 🎯 Task

Complete `/root/code/log_experiment.py` so one run in the `Default` experiment records its hyperparameters, evaluation metrics, and the sklearn model.

## 🧠 Concepts

- MLflow Tracking
- Runs and experiments
- Parameters vs metrics vs artifacts
- `mlflow.start_run()` context manager
- MLflow model flavors (`mlflow.sklearn`)

## 🔧 Commands

```python
with mlflow.start_run():
    mlflow.log_params(params)

    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("f1_score", f1)

    mlflow.sklearn.log_model(model, "model")
```

Run the script:

```bash
cd /root/code
/root/code/ml-env/bin/python log_experiment.py
```

Expected output:

```text
accuracy=0.75, f1_score=0.8571428571428571
```

## 📊 What Goes Where

| MLflow call | Stores | Example |
|---|---|---|
| `log_params(dict)` | Inputs/config (strings) | `n_estimators=100` |
| `log_metric(name, value)` | Numeric results | `accuracy=0.75` |
| `sklearn.log_model(model, path)` | Serialized model artifact | `model/MLmodel`, `model.pkl` |

## ✅ Verify

In the MLflow UI, open the latest run in `Default`:

- Parameters: `n_estimators`, `max_depth`, `random_state`
- Metrics: `accuracy`, `f1_score`
- Artifacts: `model/` folder

## ⚠️ Common Issues

- **Wrong metric name:** the variable is `f1`, but the metric must be named `f1_score`.
- **`log_param` instead of `log_params`:** `log_params` takes a whole dict.
- **`mlflow.log_model` does not exist:** use the flavor module, `mlflow.sklearn.log_model`.
- **MLflow 3.x:** `artifact_path` was renamed `name` and models appear under "Logged models". Passing the name positionally works on both versions.
- **Connection refused:** check that the tracking server is running on port 5000.
- **Script run with the wrong Python:** use the venv interpreter that has `mlflow` installed.

## 📝 Key Takeaway

**MLflow tracking separates a run into three things: parameters (what you configured), metrics (what you measured), and artifacts (what you produced).** Logging all three makes a run searchable, comparable, and reproducible.