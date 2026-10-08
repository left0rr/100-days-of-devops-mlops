# Day 6 — Cron Jobs

## 🎯 Task

Install and start `crond` on all app servers and create a scheduled cron job for the `root` user.

## 🧠 Concepts

- Cron / `crond`
- `cronie`
- `systemctl`
- `crontab`
- Scheduled tasks
- Cron timing syntax

## 🔧 Commands

### Install Cron

```bash
sudo yum install -y cronie
```

### Start and enable the service

```bash
sudo systemctl start crond
sudo systemctl enable crond
```

Check:

```bash
sudo systemctl status crond
```

### Edit root's cron jobs

```bash
sudo crontab -e
```

List root's cron jobs:

```bash
sudo crontab -l
```

## ⏰ Cron Job Syntax

A cron schedule has **5 fields**:

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week (0-7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

Example:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

Means:

```text
*/5 → every 5 minutes
*   → every hour
*   → every day of the month
*   → every month
*   → every day of the week
```

### ⭐ Common examples

```cron
* * * * *       # every minute
*/5 * * * *     # every 5 minutes
*/15 * * * *    # every 15 minutes
0 * * * *       # every hour
0 0 * * *       # every day at midnight
0 9 * * 1       # every Monday at 09:00
```

## 🧠 Important

`crond` is the **service/daemon** that runs scheduled jobs.

`crontab` is the **schedule/configuration** containing the jobs.

For a specific user:

```bash
crontab -e
```

For root:

```bash
sudo crontab -e
```

## 📝 Key Takeaway

**Cron = Linux's built-in scheduler for automatically running commands at specific times.**

Remember:

```text
minute hour day-of-month month day-of-week
```

And:

```text
*/5 * * * *
```

means **every 5 minutes**.