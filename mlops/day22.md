# MLOps Day 22 — MLflow Experiments and Tags

## 🎯 Task

Register two new MLflow experiments in the UI, each with experiment-level metadata:

- `fraud-detection`: non-empty description, tag `team=ml-platform`
- `churn-prediction`: tag `team=analytics`

`Default` and `legacy-models` must not be modified.

## 🧠 Concepts

- MLflow experiments vs runs
- Experiment-level tags
- Experiment description
- Separating projects by experiment
- Tracking server UI

## 🔧 Steps (UI)

1. Open the MLflow UI and create a new experiment named `fraud-detection`.
2. On its page, add a description and the tag `team = ml-platform`.
3. Repeat for `churn-prediction` with the tag `team = analytics`.

## 🐍 Equivalent in Python

```python
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")
client = mlflow.MlflowClient()

exp_id = client.create_experiment("fraud-detection", tags={"team": "ml-platform"})
client.set_experiment_tag(exp_id, "mlflow.note.content", "Fraud detection models")
```

## ✅ Verify

```python
for name in ["fraud-detection", "churn-prediction"]:
    print(name, client.get_experiment_by_name(name).tags)
```

## 💡 Good to Know

The UI description is stored as the experiment tag `mlflow.note.content`. Description and custom tags are the same mechanism.

## ⚠️ Common Issues

- **Run tags vs experiment tags:** the task needs tags on the experiment, not on a run.
- **Exact key and value:** `team`, `ml-platform`, `analytics`, case-sensitive.
- **Do not edit reference experiments:** `Default` and `legacy-models` are seeded for comparison.
- **`mlflow.set_experiment()` in code:** creates the experiment but adds no tags or description.

## 📝 Key Takeaway

**One experiment per project keeps runs organized, and experiment-level tags such as `team` make ownership searchable and filterable on the platform.**