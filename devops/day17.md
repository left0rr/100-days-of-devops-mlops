# DevOps Day 17 — PostgreSQL Database Setup

## Objective
Configure PostgreSQL on the Nautilus database server:
- Create user `kodekloud_roy`.
- Set the required password.
- Create database `kodekloud_db6`.
- Grant the user all database-level privileges.
- Do not restart PostgreSQL.

## 1. Connect to the database server

From the jump host:

```bash
ssh peter@stdb01
```

Switch to the PostgreSQL operating-system account:

```bash
sudo -i -u postgres
```

Open the PostgreSQL interactive terminal:

```bash
psql
```

**Important:** The Linux account is `postgres`, not `pgsql` or `psgql`. The PostgreSQL service is not necessarily named `pgsql`; inspect available services if needed:

```bash
systemctl list-units --type=service | grep -i postgres
systemctl list-unit-files | grep -i postgres
```

## 2. Create a database user

```sql
CREATE USER kodekloud_roy WITH PASSWORD 'your_password';
```

PostgreSQL users are roles with login permission. `CREATE USER` is effectively shorthand for creating a login role.

If the user already exists and only the password needs correcting:

```sql
ALTER USER kodekloud_roy WITH PASSWORD 'your_password';
```

## 3. Create a database

```sql
CREATE DATABASE kodekloud_db6;
```

## 4. Grant database privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db6 TO kodekloud_roy;
```

This grants database-level privileges, including CONNECT, CREATE, and TEMPORARY.

## 5. Verify the configuration

List databases:

```sql
\l
```

List users and roles:

```sql
\du
```

Inspect a particular database:

```sql
\l kodekloud_db6
```

The database access privileges should include `kodekloud_roy`.

## 6. Exit cleanly

Exit PostgreSQL:

```sql
\q
```

Return from the PostgreSQL Linux account:

```bash
exit
```

## Essential PostgreSQL commands

| Command | Purpose |
|---|---|
| `psql` | Open PostgreSQL shell |
| `\l` | List databases |
| `\du` | List roles/users |
| `\c database_name` | Connect to a database |
| `\dt` | List tables |
| `\q` | Quit PostgreSQL |
| `CREATE USER` | Create a login role |
| `ALTER USER` | Change user attributes/password |
| `CREATE DATABASE` | Create a database |
| `GRANT` | Assign privileges |
| `REVOKE` | Remove privileges |

## Troubleshooting lessons

- **Missing semicolon:** PostgreSQL displays `postgres-#` when a statement is incomplete. Finish the statement with `;`. Use `Ctrl+C` to cancel an unfinished command.
- **User already exists:** Use `ALTER USER` to correct its password instead of creating it again.
- **Database does not exist:** Create it before running `GRANT`.
- **Service name not found:** Inspect installed service units instead of guessing the service name.
- **Password correction:** `ALTER USER username WITH PASSWORD 'new_password';` takes effect immediately.

## Key concepts to remember

- PostgreSQL is a relational database management system.
- A role can represent a user or a group of permissions.
- A database contains schemas, tables, sequences, and other objects.
- `GRANT ALL PRIVILEGES ON DATABASE` grants database-level privileges; it does not automatically grant all privileges on every table inside the database.
- User, database, and privilege changes take effect without restarting PostgreSQL.

**Operational rule:** Never restart a database service unnecessarily, especially on a shared or production system.