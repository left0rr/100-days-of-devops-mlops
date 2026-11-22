# DevOps Day 19 — Hosting Multiple Static Websites with Apache

## Objective

Configure Apache (`httpd`) on App Server 2 to host two static websites on port `8089`:

- `/official/` → `/var/www/html/official/`
- `/cluster/` → `/var/www/html/cluster/`

The focus is **path-based URL mapping using Apache Alias directives**.

## 1. Connect and install Apache

From the jump host:

```bash
ssh steve@stapp02
```

Install Apache:

```bash
sudo yum install -y httpd
```

Check the installed version:

```bash
httpd -v
```

## 2. Transfer website backups

The backup directories are on the jump host. Run these commands there:

```bash
scp -r /home/thor/official steve@stapp02:/tmp/
scp -r /home/thor/cluster steve@stapp02:/tmp/
```

On App Server 2, prepare the document directories and copy their contents:

```bash
sudo mkdir -p /var/www/html/official
sudo mkdir -p /var/www/html/cluster

sudo cp -r /tmp/official/. /var/www/html/official/
sudo cp -r /tmp/cluster/. /var/www/html/cluster/
```

The `/.` copies the directory contents, including hidden files, without introducing an unwanted nested directory.

## 3. Configure Apache

Edit the main configuration:

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

Change the listening port:

```apache
Listen 8089
```

Add the following at the bottom of the file, outside any `<VirtualHost>` block:

```apache
Alias /official/ "/var/www/html/official/"
Alias /cluster/ "/var/www/html/cluster/"

<Directory "/var/www/html/official">
    Require all granted
    AllowOverride All
</Directory>

<Directory "/var/www/html/cluster">
    Require all granted
    AllowOverride All
</Directory>
```

### Apache directive concepts

| Directive | Purpose |
|---|---|
| `Listen 8089` | Makes Apache listen on port 8089 |
| `Alias` | Maps a URL path to a filesystem directory |
| `<Directory>` | Defines access rules for a filesystem directory |
| `Require all granted` | Allows requests to access the directory |
| `AllowOverride All` | Allows `.htaccess` files to override permitted settings |

**Important:** `Alias` is path-based mapping. It allows multiple websites to share one Apache server and port without requiring separate virtual hosts.

## 4. Validate and activate the configuration

Check Apache syntax:

```bash
sudo httpd -t
```

Expected:

```text
Syntax OK
```

Enable and start Apache:

```bash
sudo systemctl enable --now httpd
```

If Apache is already running and the configuration changes:

```bash
sudo systemctl reload httpd
```

Check service status:

```bash
sudo systemctl status httpd
```

## 5. Verify both websites

Run on App Server 2:

```bash
curl http://localhost:8089/official/
curl http://localhost:8089/cluster/
```

Each URL should return the content of its corresponding website.

## Troubleshooting reference

| Symptom | Diagnostic command |
|---|---|
| Apache fails to start | `sudo httpd -t` |
| Port conflict | `sudo ss -lntp \| grep 8089` |
| HTTP 403 | `ls -ld /var/www/html /var/www/html/official /var/www/html/cluster` |
| HTTP 404 | `ls -la /var/www/html/official /var/www/html/cluster` |
| Apache errors | `sudo tail -f /var/log/httpd/error_log` |
| Local request fails | `curl -v http://localhost:8089/official/` |

For HTTP 403, check filesystem traversal/read permissions and SELinux contexts if SELinux is enforcing.

## Key DevOps concepts

- **Path-based routing:** Different URL prefixes serve different website directories.
- **Document root vs Alias:** `DocumentRoot` defines the default website directory; `Alias` maps additional URL paths to other directories.
- **Apache access control:** `<Directory>` controls access to filesystem locations.
- **Configuration validation:** Always run `httpd -t` before reloading or restarting.
- **Deployment workflow:** Transfer artifacts, place them in the document directories, configure mappings, validate, activate, and test.
- **Production considerations:** Use least-privilege filesystem permissions, appropriate SELinux contexts, HTTPS, and monitoring.

**Operational takeaway:** Apache `Alias` is useful when several static sites must be hosted under different URL paths on one server and one listening port.