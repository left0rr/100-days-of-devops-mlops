# DevOps Day 20 — Nginx + PHP-FPM Integration

## Objective
Configure Nginx on App Server 2 (`stapp02`) to serve PHP applications from `/var/www/html` on port `8096`, using PHP-FPM 8.3 over a Unix socket.

## 1. Install PHP 8.3 and PHP-FPM

On CentOS Stream 9, enable the required PHP module stream:

```bash
sudo dnf module list php
sudo dnf module enable php:8.3 -y
sudo dnf install php php-fpm -y
```

Verify versions:

```bash
php -v
php-fpm -v
```

**Concept:** DNF module streams allow selection of a specific application version when multiple versions are available.

## 2. Configure the PHP-FPM pool

Edit:

```bash
sudo vi /etc/php-fpm.d/www.conf
```

Set the Unix socket:

```ini
listen = /var/run/php-fpm/default.sock
```

The existing ACL included Nginx:

```ini
listen.acl_users = apache,nginx
```

Create the socket directory if needed and start PHP-FPM:

```bash
sudo mkdir -p /var/run/php-fpm
sudo php-fpm -t
sudo systemctl enable --now php-fpm
```

Verify:

```bash
sudo systemctl status php-fpm --no-pager
ls -l /var/run/php-fpm/default.sock
```

**Concepts:**
- PHP-FPM (FastCGI Process Manager) executes PHP scripts outside Nginx.
- A Unix domain socket provides local inter-process communication without using a TCP port.
- Socket permissions and ACLs determine which processes can connect.
- `/var/run` commonly points to `/run`; runtime socket files are not persistent across reboots.

## 3. Configure Nginx FastCGI integration

Edit the existing configuration:

```bash
sudo vi /etc/nginx/conf.d/php-fpm.conf
```

Configuration:

```nginx
upstream php-fpm {
    server unix:/var/run/php-fpm/default.sock;
}

server {
    listen 8096;
    server_name _;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        try_files $uri =404;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass php-fpm;
    }
}
```

**Important directives:**
- `root`: filesystem location of application files.
- `index`: default file when a directory is requested.
- `location ~ \.php$`: matches PHP requests.
- `SCRIPT_FILENAME`: passes the full PHP script path to PHP-FPM.
- `fastcgi_pass`: forwards PHP requests to the configured upstream socket.
- `try_files`: prevents execution of nonexistent PHP scripts.

## 4. Validate and start Nginx

```bash
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
```

## 5. Test from the jump host

```bash
curl http://stapp02:8096/index.php
```

Expected application response:

```text
Welcome to xFusionCorp Industries!
```

## Troubleshooting reference

| Symptom | What to inspect |
|---|---|
| Connection refused | Nginx service and listening port |
| 502 Bad Gateway | PHP-FPM status, socket path and ACL |
| 404 Not Found | Document root, file existence, `try_files` |
| PHP source displayed | FastCGI location and `fastcgi_pass` |

Useful diagnostics:

```bash
sudo nginx -t
sudo php-fpm -t
sudo journalctl -u php-fpm -n 50 --no-pager
sudo tail -n 50 /var/log/nginx/error.log
```

## Key takeaway

Nginx handles incoming HTTP requests and serves static files. For PHP requests, it acts as a FastCGI client and forwards execution to PHP-FPM through a Unix socket. PHP-FPM executes the script and returns the response to Nginx.

