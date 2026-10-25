# MLOps Day 12 — DVC Remote Configuration with SeaweedFS

## 1. Configure an S3-compatible remote

DVC can use S3-compatible object stores, not just AWS S3. SeaweedFS exposes an S3 API endpoint.

```bash
# List configured remotes and their URLs
dvc remote list

# Create/replace remote and set it as default
dvc remote add -d -f s3 s3://dvc-storage

# Configure endpoint and credentials
dvc remote modify s3 endpointurl http://localhost:8333
dvc remote modify s3 access_key_id weedadmin
dvc remote modify s3 secret_access_key weedadmin123
```

**Configuration meanings:**
- `s3` — DVC's local name for this remote.
- `s3://dvc-storage` — bucket address; the `s3://` scheme selects the S3 backend.
- `endpointurl` — directs S3 requests to SeaweedFS rather than AWS.
- `access_key_id` / `secret_access_key` — credentials used to authenticate.
- `-d` — selects the default remote.
- `-f` — overwrites an existing remote with the same name.

## 2. Remote management reference

```bash
# Set an existing remote as default
dvc remote default s3

# Change the remote's bucket URL
dvc remote modify s3 url s3://another-bucket

# Inspect project configuration
cat .dvc/config

# Show detailed command errors
dvc push -v
```

Use `dvc remote modify` when the remote already exists. Use `dvc remote add -d -f` when you need to replace its URL and set it as default in one step.

## 3. Credentials and configuration scope

```bash
# Store a setting in local, unshared configuration
dvc remote modify --local s3 secret_access_key YOUR_SECRET
```

- Project settings are normally stored in `.dvc/config`.
- Local settings go in `.dvc/config.local` and are intended for machine-specific values or secrets.
- Avoid committing real credentials to shared project configuration. Use the lab-provided credentials only for the lab.

## 4. Push verification and troubleshooting

After configuring the remote:

```bash
dvc push
```

Then inspect the SeaweedFS Filer UI at:

`/buckets/dvc-storage/files/md5/`

DVC stores content-addressed objects under hashed paths, so you should see objects beneath `files/md5/`, not necessarily a file named `transactions.csv`.

**Common failures:**
- **No remote specified:** set the default with `dvc remote default s3`.
- **Remote already exists:** use `dvc remote modify` or `dvc remote add -d -f`.
- **Connection refused:** confirm SeaweedFS is running and the endpoint is reachable from the environment running DVC.
- **Access denied:** check the access key and secret.
- **Wrong bucket or no objects:** confirm the remote URL and verify that `dvc push` completed successfully.

**Key takeaway:** For S3-compatible storage, configure the bucket URL, endpoint, credentials, and default remote correctly; then verify that DVC objects actually appear in the object store.