# MLOps Day 14 — Build a Reproducible DVC Pipeline

## 1. Define the Pipeline

Project: `/root/code/fraud-detection`

Create `dvc.yaml` with two stages:

```yaml
stages:
  process_data:
    cmd: python3 src/data/process_data.py
    deps:
      - data/raw/transactions.csv
      - src/data/process_data.py
    outs:
      - data/processed/clean_transactions.csv

  split_data:
    cmd: python3 src/data/split_data.py
    deps:
      - data/processed/clean_transactions.csv
      - src/data/split_data.py
    outs:
      - data/processed/train.csv
      - data/processed/test.csv
```

**Key concepts:**
- `cmd`: command executed by the stage.
- `deps`: input data and scripts DVC monitors for changes.
- `outs`: files generated and tracked by DVC.
- `dvc.yaml`: defines the pipeline stages and dependencies.

## 2. Create Stages with the CLI

Alternative to manually editing YAML:

```bash
dvc stage add -n process_data \
  -d data/raw/transactions.csv \
  -d src/data/process_data.py \
  -o data/processed/clean_transactions.csv \
  python3 src/data/process_data.py
```

```bash
dvc stage add -n split_data \
  -d src/data/split_data.py \
  -d data/processed/clean_transactions.csv \
  -o data/processed/train.csv \
  -o data/processed/test.csv \
  python3 src/data/split_data.py
```

Use `--force` to overwrite an existing stage definition:

```bash
dvc stage add --force ...
```

**Syntax reminder:** Put the command to execute at the end. When using `\` for multiline commands, it must be the final character on the line.

## 3. Understand the Stage Dependency Chain

```text
transactions.csv
       |
       v
  process_data
       |
       v
clean_transactions.csv
       |
       v
   split_data
     /    \
    v      v
 train.csv test.csv
```

Because `split_data` depends on the output of `process_data`, DVC automatically establishes the execution order.

## 4. Reproduce and Verify

```bash
dvc repro
dvc dag
dvc dag --outs
dvc dag --dot
dvc status
ls -l data/processed/
```

- `dvc repro`: runs stages in dependency order when required and creates or updates `dvc.lock`.
- `dvc dag`: displays the stage dependency graph.
- `dvc dag --outs`: displays the graph with outputs.
- `dvc dag --dot`: prints the graph in DOT format.
- `dvc status`: checks whether data and pipeline stages are up to date.

## 5. Lock File and Reproducibility

- `dvc.yaml` defines commands, dependencies, and outputs.
- `dvc.lock` records the resolved stage information and dependency/output hashes after reproduction.
- DVC uses this metadata to detect changes and determine whether stages need to run again.

**Successful challenge verification:**
- `process_data` processed 15 rows.
- `split_data` produced 12 training rows and 3 test rows.
- `dvc.lock` was generated and updated.
- `dvc status` reported: `Data and pipelines are up to date.`

## Key Takeaway

A DVC pipeline makes data processing reproducible by defining each stage's command, inputs, and outputs. Connecting one stage's output to the next stage's dependency lets DVC build the execution graph, run stages in order, and track whether results are stale.