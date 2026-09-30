# Day 1 — Linux User Management

## 🎯 Task

Create a user `siva` with a non-interactive shell
on App Server 2.

## 🧠 Concepts

- Linux users
- Service accounts
- Interactive vs non-interactive shells
- SSH / jump hosts
- Principle of least privilege

## 🔧 Commands

```bash
ssh steve@stapp02

sudo useradd -s /usr/sbin/nologin siva

getent passwd siva
