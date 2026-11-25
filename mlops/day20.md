# MLOps Day 20 — MLflow Tracking Server Setup

## Objective
Deploy a local MLflow Tracking Server on the controlplane, allowing the ML team to track experiments and store metadata and artifacts.

## 1. Prepare storage directories

```bash
mkdir -p /root/code/mlflow-backend
mkdir -p /root/code/mlflow-artifacts
ls -ld /root/code/mlflow-backend /root/code/mlflow-artifacts
```

**Concept — Separate storage responsibilities:**
- **Backend store:** Stores experiment metadata, runs, parameters, metrics, and tags.
- **Artifact store:** Stores larger files such as models, datasets, plots, and other run artifacts.

## 2. Check MLflow version and supported options

```bash
mlflow --version
mlflow server --help
```

Useful options to identify:
- `--host`
- `--port`
- `--backend-store-uri`
- `--default-artifact-root`
- `--allowed-hosts`
- `--cors-allowed-origins`

**Concept:** Always inspect the installed CLI version and its available flags before deploying. Security options can vary between MLflow versions.

## 3. Launch the tracking server in the background

```bash
nohup mlflow server \
  --host 0.0.0.0 \
  --port 5000 \
  --backend-store-uri sqlite:////root/code/mlflow-backend/mlflow.db \
  --default-artifact-root file:///root/code/mlflow-artifacts/ \
  --allowed-hosts "*" \
  --cors-allowed-origins "*" \
  > /root/code/mlflow-server.log 2>&1 &
```

### Important flags explained

| Option | Purpose |
|---|---|
| `--host 0.0.0.0` | Listen on all network interfaces, not just localhost |
| `--port 5000` | Set the tracking server port |
| `--backend-store-uri` | Define the SQLite metadata database |
| `--default-artifact-root` | Define where run artifacts are stored |
| `--allowed-hosts "*"` | Accept requests with any Host header |
| `--cors-allowed-origins "*"` | Allow requests from any browser origin |
| `nohup` | Keep the process running after terminal closure |
| `&` | Run the process in the background |
| `> logfile 2>&1` | Redirect standard output and errors to a log file |

**Important URI detail:**

```text
sqlite:////root/code/mlflow-backend/mlflow.db
```

Four slashes after `sqlite:` indicate an absolute filesystem path.

```text
file:///root/code/mlflow-artifacts/
```

Three slashes after `file:` indicate an absolute local artifact directory.

## 4. Verify the tracking server

**Check the process:**

```bash
pgrep -af "mlflow server"
```

**Inspect server logs:**

```bash
tail -n 30 /root/code/mlflow-server.log
```

**Check HTTP availability:**

```bash
curl -I http://127.0.0.1:5000/
```

Expected response:

```text
HTTP/1.1 200 OK
```

**Check listening port, if `ss` is available:**

```bash
ss -lntp | grep 5000
```

## 5. Understand proxy and browser security

Two separate controls matter when MLflow is accessed through a lab proxy:

- **Allowed hosts:** Validates the HTTP `Host` header. A proxy may send a hostname different from `localhost`.
- **CORS allowed origins:** Controls which browser origins may make cross-origin requests.

For this lab, both were configured with `"*"` to satisfy the proxy requirements.

**Security note:** Wildcard host and origin policies are convenient for isolated labs, but should not be copied blindly into production. Use explicit trusted hostnames and origins in production. Wildcard CORS also disables credentialed cross-origin requests.

## 6. Troubleshooting reference

| Problem | What to investigate |
|---|---|
| Server exits immediately | Check log file and backend directory |
| Database startup error | Verify SQLite path and parent directory |
| UI returns 403 | Check `--allowed-hosts` |
| Browser CORS error | Check `--cors-allowed-origins` |
| Cannot access remotely | Check `--host 0.0.0.0`, port and network rules |
| Server not running | Check `pgrep`, logs and process status |

Useful commands:

```bash
pgrep -af "mlflow server"
tail -f /root/code/mlflow-server.log
```
