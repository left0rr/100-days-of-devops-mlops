# Day 2 — Temporary User Account

## 🎯 Task

Create a user javed with an expiry date of 2027-03-28 on App Server 3.

## 🧠 Concepts

- Linux user accounts
- Temporary users
- Account expiration
- useradd options
- SSH / jump hosts

## 🔧 Commands

```bash
ssh <username>@stapp03

sudo useradd -e 2027-03-28 javed

sudo chage -l javed

getent passwd javed