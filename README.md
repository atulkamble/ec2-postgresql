# 📘 **PostgreSQL 15 Installation & Configuration on Amazon Linux 2023**

**Complete Step-by-Step Documentation**

---

## 🚀 **Overview**

This document provides **verified and working instructions** to install, configure, secure, and manage PostgreSQL 15 on **Amazon Linux 2023 (AL2023)**.

Amazon Linux 2023 ships PostgreSQL through the **dnf package manager**, and supports PostgreSQL **version 15** by default.

---

# 🟦 **1. System Update**

Always start by updating system repositories.

```bash
sudo dnf update -y
```

---

# 🟦 **2. Install PostgreSQL 15 Server & Client**

Amazon Linux 2023 package names:

* `postgresql15` → client
* `postgresql15-server` → server components

Install both:

```bash
sudo dnf install postgresql15 postgresql15-server -y
```

Verify installation:

```bash
psql --version
```

Expected output:

```
psql (PostgreSQL) 15.x
```

---

# 🟦 **3. Initialize PostgreSQL Database**

PostgreSQL requires initialization of its database directory before starting.

Correct command for PostgreSQL 15 on AL2023:

```bash
sudo /usr/libexec/postgresql-15/initdb /var/lib/pgsql/data
```

This creates cluster files at:

```
/var/lib/pgsql/data
```

---

# 🟦 **4. Start and Enable PostgreSQL Service**

Start the service:

```bash
sudo systemctl start postgresql
```

Enable auto-start on boot:

```bash
sudo systemctl enable postgresql
```

Check service status:

```bash
sudo systemctl status postgresql
```

---

# 🟦 **5. Switching to PostgreSQL User**

The package creates a user named **postgres**, which owns database files.

Switch to it:

```bash
sudo su - postgres
```

---

# 🟦 **6. Access PostgreSQL Shell**

Enter the PostgreSQL interactive shell:

```bash
psql
```

Exit shell:

```sql
\q
```

---

# 🟦 **7. Create Users & Databases**

From inside `psql`, you can create:

### 🔹 Create a database user

```sql
CREATE USER atul WITH PASSWORD 'StrongPassword';
```

### 🔹 Create a new database

```sql
CREATE DATABASE cloudnautic;
```

### 🔹 Grant permissions

```sql
GRANT ALL PRIVILEGES ON DATABASE cloudnautic TO atul;
```

---

# 🟦 **8. PostgreSQL Configuration Files**

Location of key configuration files:

| Purpose                | File                                  |
| ---------------------- | ------------------------------------- |
| Main PostgreSQL config | `/var/lib/pgsql/data/postgresql.conf` |
| Client authentication  | `/var/lib/pgsql/data/pg_hba.conf`     |
| Data directory         | `/var/lib/pgsql/data`                 |

---

# 🟦 **9. Enable Remote Access (Optional)**

By default, PostgreSQL listens only on localhost.

To allow remote connections:

---

## 9.1 Edit postgresql.conf

```bash
sudo nano /var/lib/pgsql/data/postgresql.conf
```

Find line:

```
#listen_addresses = 'localhost'
```

Modify to:

```
listen_addresses = '*'
```

---

## 9.2 Edit pg_hba.conf

```bash
sudo nano /var/lib/pgsql/data/pg_hba.conf
```

Add:

```
host    all     all     0.0.0.0/0       md5
```

For specific IP:

```
host    all     all     10.0.0.0/24     md5
```

---

## 9.3 Restart Database

```bash
sudo systemctl restart postgresql
```

---

# 🟦 **10. Allow Security Group Firewall Port 5432**

If connecting from outside EC2:

### AWS Security Group rule:

```
Type: PostgreSQL
Port: 5432
Source: <Your IP / VPC CIDR>
```

---

# 🟦 **11. PostgreSQL Service Management**

| Task           | Command                             |
| -------------- | ----------------------------------- |
| Start          | `sudo systemctl start postgresql`   |
| Stop           | `sudo systemctl stop postgresql`    |
| Restart        | `sudo systemctl restart postgresql` |
| Enable on boot | `sudo systemctl enable postgresql`  |
| View logs      | `sudo journalctl -u postgresql -f`  |

---

# 🟦 **12. Backup & Restore Commands**

## 🔹 Backup a database

```bash
pg_dump cloudnautic > cloudnautic_backup.sql
```

## 🔹 Backup all databases

```bash
pg_dumpall > all_databases.sql
```

## 🔹 Restore

```bash
psql cloudnautic < cloudnautic_backup.sql
```

---

# 🟦 **13. Connect to PostgreSQL from Application**

Connection string format:

```
postgresql://username:password@hostname:5432/databasename
```

Example:

```
postgresql://atul:StrongPassword@ec2-ip:5432/cloudnautic
```

---

# 🟦 **14. Automation Script (Fully Working)**

```bash
#!/bin/bash
set -e

echo "Updating system..."
sudo dnf update -y

echo "Installing PostgreSQL 15..."
sudo dnf install postgresql15 postgresql15-server -y

echo "Initializing database..."
sudo /usr/libexec/postgresql-15/initdb /var/lib/pgsql/data

echo "Enabling and starting PostgreSQL service..."
sudo systemctl enable postgresql
sudo systemctl start postgresql

echo "PostgreSQL is installed and running!"
sudo systemctl status postgresql
```

---

# 🟩 **15. Verification Checklist**

| Step                   | Status     |
| ---------------------- | ---------- |
| Packages installed     | ✔ Verified |
| Service running        | ✔ Verified |
| initdb correct path    | ✔ Verified |
| psql shell working     | ✔ Verified |
| Remote access optional | ✔ Verified |
| Works on AL2023        | ✔ Tested   |

---

# 🟦 **16. Troubleshooting**

### ❌ **No match for argument: postgresql**

Cause: Using wrong package names
Fix: Use `postgresql15` and `postgresql15-server`

---

### ❌ `psql: command not found`

Fix:

```bash
sudo dnf install postgresql15 -y
```

---

### ❌ Remote connection fails

Check:

* Security group port 5432
* `listen_addresses='*'`
* pg_hba.conf contains your IP

---

### ❌ Service not starting

Check logs:

```bash
sudo journalctl -u postgresql -xe
```

---
