# DevOps Day 11 — Install and Configure Tomcat

## 1. Task
Install Apache Tomcat on an application server, configure it to listen on port `3001`, deploy a `ROOT.war` web application, and verify that it responds over HTTP.

## 2. What Is Tomcat?

**Apache Tomcat** is an open-source Java web server and servlet container. It runs Java web applications packaged as WAR files and handles HTTP requests by passing them to the deployed application.

**Why use it?**
- Hosts Java web applications.
- Processes HTTP requests and returns application responses.
- Supports Java Servlet and JSP technologies.
- Provides a service that can be managed with `systemctl`.

Tomcat is commonly used to deploy Java web apps without needing a full Java EE application server.

## 3. Challenge Workflow

### Step 1 — Install Tomcat
On the target application server:

```bash
sudo yum install -y tomcat
```

Check the service:

```bash
sudo systemctl status tomcat
```

**Why:** Installs Tomcat and checks whether its service is running. After installation, it may initially be inactive.

### Step 2 — Configure the HTTP port

Edit Tomcat's main configuration file:

```bash
sudo vi /etc/tomcat/server.xml
```

Find the HTTP `Connector` and change its port from `8080` to `3001`, preserving the other connector settings.

**Why:** Tomcat's HTTP connector determines which port accepts web requests.

### Step 3 — Transfer the WAR file

From the Jump host, copy the application to the target server:

```bash
scp /tmp/ROOT.war tony@stapp01:/tmp/ROOT.war
```

**Why:** `scp` securely transfers files over SSH.

### Step 4 — Deploy the application

On the target server:

```bash
sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/ROOT.war
```

**Why:** Tomcat deploys WAR files placed in its configured application deployment directory.

**Important naming rule:** `ROOT.war` is deployed at the root context, so the app is accessed at `http://stapp01:3001/`. A file named `myapp.war` would normally be available at `/myapp/`.

### Step 5 — Start Tomcat and enable it at boot

```bash
sudo systemctl enable --now tomcat
sudo systemctl restart tomcat
sudo systemctl status tomcat
```

- `enable` — starts Tomcat automatically on future boots.
- `--now` — starts the service immediately.
- `restart` — restarts the service to apply configuration changes.
- `status` — confirms service state.

### Step 6 — Verify the application

From the Jump host:

```bash
curl http://stapp01:3001
```

A successful response containing the application's HTML confirms that Tomcat is reachable and serving the deployed application.

## 4. Useful Tomcat Reference

| Command | Purpose |
|---|---|
| `sudo systemctl start tomcat` | Start Tomcat |
| `sudo systemctl stop tomcat` | Stop Tomcat |
| `sudo systemctl restart tomcat` | Restart after configuration changes |
| `sudo systemctl enable tomcat` | Enable startup at boot |
| `sudo systemctl status tomcat` | Check service health |
| `sudo journalctl -u tomcat --no-pager -n 50` | Inspect recent service logs |
| `sudo grep -n 'Connector port=' /etc/tomcat/server.xml` | Check configured connector ports |
| `curl http://stapp01:3001` | Test HTTP response from the Jump host |

If `ss` is installed, `sudo ss -lntp | grep ':3001'` can help confirm that a process is listening on port 3001. If it is unavailable, use `curl` and the Tomcat logs to troubleshoot.

## 5. Common Issues

- **Connection refused:** Check that Tomcat is running and the connector uses the correct port.
- **Application not found:** Confirm the WAR file is in the configured deployment directory and is named `ROOT.war` for the root URL.
- **Configuration changes not applied:** Restart Tomcat and inspect its logs.
- **Deployment errors:** Check file permissions and logs for WAR extraction or application startup errors.

## Key Takeaway

Tomcat deployment follows a repeatable pattern: **install → configure the connector → transfer the WAR → deploy → start the service → verify with HTTP**. Always check both the service status and the application's actual HTTP response.