# MLOps Day 13 — Fix DVC Pull Credentials

**Goal:** Restore a missing dataset from SeaweedFS after cloning a DVC project.

### 1. Diagnose the issue
```bash
cd /root/code/fraud-detection
dvc pull
cat .dvc/config
```
**Why:** Check the pull error and inspect the DVC remote configuration. The pointer file can exist even when the actual dataset is missing.

### 2. Configure the S3 remote
```bash
dvc remote modify s3 endpointurl http://localhost:8333
dvc remote modify s3 access_key_id weedadmin
dvc remote modify s3 secret_access_key weedadmin123
```
**Why:** Configure the SeaweedFS endpoint and the credentials needed to authenticate with its S3-compatible API.

### 3. Pull and verify the dataset
```bash
dvc pull
md5sum data/raw/transactions.csv
cat data/raw/transactions.csv.dvc
dvc status
```
**Why:**
- `dvc pull` downloads and restores the missing dataset.
- `md5sum` checks the file's content hash against the pointer's `md5` value.
- `dvc status` checks whether the workspace data matches the tracked metadata.

### Key Takeaway
When `dvc pull` fails, inspect `.dvc/config`, verify the endpoint and credentials, retry the pull, and compare the downloaded file's hash with the DVC pointer. Never commit real credentials to version control.