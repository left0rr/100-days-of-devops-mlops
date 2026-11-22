# DevOps Day 18 — MariaDB Database Server Setup

## Objective

Configure MariaDB on the Nautilus DB server:

- Install and start MariaDB.
- Create database `kodekloud_db6`.
- Create user `kodekloud_aim` with a password.
- Grant full privileges on the database to that user.
- Verify the configuration.

## 1. Connect to the database server

From the jump host:

```bash
ssh peter@stdb01
```

Use the credentials provided by the challenge.

## 2. Install MariaDB

On the database server:

```bash
sudo yum install -y mariadb-server
```

Check service status:

```bash
sudo systemctl status mariadb
```

Enable MariaDB at boot and start it:

```bash
sudo systemctl enable --now mariadb
```

Useful service commands:

```bash
sudo systemctl start mariadb
sudo systemctl stop mariadb
sudo systemctl restart mariadb
sudo systemctl status mariadb
```

**Concept:** `enable` configures automatic startup at boot; `start` starts the service immediately. `enable --now` does both.

## 3. Access the MariaDB shell

```bash
sudo mysql
```

Alternative:

```bash
sudo mariadb
```

Expected prompt:

```text
MariaDB [(none)]>
```

## 4. Create the database

```sql
CREATE DATABASE kodekloud_db6;
```

List databases:

```sql
SHOW DATABASES;
```

## 5. Create a database user

```sql
CREATE USER 'kodekloud_aim'@'%' IDENTIFIED BY 'your_password';
```

**Concept:**
- `kodekloud_aim` is the database account.
- `%` allows connections from any host.
- `localhost` restricts connections to the local machine.

For production, prefer restricting the host to the application's trusted server or network.

## 6. Grant database privileges

```sql
GRANT ALL PRIVILEGES ON kodekloud_db6.* TO 'kodekloud_aim'@'%';
```

Apply privilege changes:

```sql
FLUSH PRIVILEGES;
```

**Important concepts:**

- `ALL PRIVILEGES` grants all applicable privileges.
- `kodekloud_db6.*` means all tables and objects within that database.
- `*.*` means all databases and tables globally. Avoid this unless explicitly required.
- `USAGE ON *.*` is a normal account entry, not an additional global permission grant.

## 7. Verify the configuration

List databases:

```sql
SHOW DATABASES;
```

List database users and allowed hosts:

```sql
SELECT User, Host FROM mysql.user;
```

Inspect the user's permissions:

```sql
SHOW GRANTS FOR 'kodekloud_aim'@'%';
```

Expected important grant:

```sql
GRANT ALL PRIVILEGES ON `kodekloud_db6`.* TO `kodekloud_aim`@`%`
```

Exit MariaDB:

```sql
EXIT;
```

## Essential command reference

| Command | Purpose |
|---|---|
| `yum install -y mariadb-server` | Install MariaDB server |
| `systemctl enable --now mariadb` | Enable and start service |
| `sudo mysql` | Open MariaDB shell |
| `SHOW DATABASES;` | List databases |
| `CREATE DATABASE` | Create a database |
| `CREATE USER` | Create a database account |
| `GRANT ALL PRIVILEGES` | Grant permissions |
| `FLUSH PRIVILEGES` | Reload privilege tables |
| `SHOW GRANTS` | Inspect account permissions |
| `EXIT;` | Leave MariaDB shell |

## Troubleshooting and operational lessons

**1. Service not found**

Check installed packages and available service names:

```bash
rpm -qa | grep -i mariadb
systemctl list-unit-files | grep -i maria
```

**2. User already exists**

Change its password instead of recreating it:

```sql
ALTER USER 'kodekloud_aim'@'%' IDENTIFIED BY 'new_password';
```

**3. Incorrect privileges**

Inspect first:

```sql
SHOW GRANTS FOR 'kodekloud_aim'@'%';
```

Remove an unwanted database-level grant if necessary:

```sql
REVOKE ALL PRIVILEGES ON kodekloud_db6.* FROM 'kodekloud_aim'@'%';
```

Only revoke privileges that are actually unwanted.

**4. SQL syntax errors**

- SQL statements must end with `;`.
- If the prompt changes to `->`, MariaDB is waiting for the statement to finish.
- Avoid inserting stray characters or punctuation.

## DevOps concepts to remember

- **Database server:** Hosts and manages persistent application data.
- **Database account:** Controls who can authenticate.
- **Host-based access:** Determines where an account can connect from.
- **Least privilege:** Give applications only the permissions they need.
- **Service management:** Use systemd to install, start, stop, enable, and monitor services.
- **Security:** Avoid global grants, restrict network access, protect credentials, and use backups.

**Production relevance:** These skills are essential when provisioning databases for applications, CI/CD environments, containerized services, and distributed systems.