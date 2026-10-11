# DevOps Day 9 — Fix MariaDB Service

## Problem
The Nautilus application could not connect to its database because MariaDB was down on `stdb01`.

## Diagnosis
```bash
sudo systemctl status mariadb
sudo journalctl -u mariadb -n 50 --no-pager
```

The logs reported that MariaDB was not initialized because `/var/lib/mysql` was not empty. Further inspection revealed that the configured data directory was missing, while `/var/lib/mysqld` existed separately.

## Fix
```bash
sudo mkdir -p /var/lib/mysql
sudo chown mysql:mysql /var/lib/mysql
sudo chmod 755 /var/lib/mysql

sudo systemctl start mariadb
sudo systemctl enable mariadb
```

## Verification
```bash
sudo systemctl status mariadb --no-pager
sudo systemctl is-active mariadb
sudo systemctl is-enabled mariadb
```

Expected:
- Service status: `active (running)`
- Active state: `active`
- Boot configuration: `enabled`

## Key takeaways
- `systemctl status` checks the service state.
- `journalctl` reveals why a service failed.
- `mkdir -p` creates a missing directory.
- `chown mysql:mysql` assigns ownership to MariaDB's service account.
- `systemctl start` starts a service now; `systemctl enable` configures it to start at boot.
- Check the configured data directory before changing database files.

**Result:** MariaDB successfully started and was confirmed active and enabled.
