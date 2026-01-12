# Automated Backup to AWS S3 using Shell Script & Cron

> A beginner‑friendly DevOps mini‑project that demonstrates **Linux shell scripting**, **AWS IAM**, **AWS CLI**, **S3**, and **cron jobs** — all stitched together into a real, production‑style workflow.

---

## 🚀 Project Overview

This project automates the process of:

1. Creating a compressed backup of a local directory
2. Uploading (syncing) that backup to an AWS S3 bucket
3. Running the backup automatically using `cron`

The goal is **not just automation**, but also learning how real DevOps scripts are written:

* Validation
* Logging
* Error handling
* Cron‑safe execution

---

## 🧠 What I Learned

* Writing safe Bash scripts (`set -e`, validation, logging)
* Why cron jobs fail without proper `PATH`
* How AWS CLI authentication actually works
* IAM least‑privilege thinking (backup‑only user)
* How backups are handled in real systems

---

## 🛠 Tech Stack

* Bash / Shell Scripting
* AWS S3
* AWS IAM
* AWS CLI
* Linux (cron)

---

## 📂 Project Flow

```text
Local Folder
   ↓ (zip)
Compressed Backup
   ↓ (aws s3 sync)
S3 Bucket
```

---

## 🔐 AWS Setup (One‑Time)

### 1️⃣ Create S3 Bucket

```bash
aws s3 mb s3://local-backup-1201
```

### 2️⃣ Create IAM Backup User

* Programmatic access only
* Attach policy with:

  * `s3:PutObject`
  * `s3:ListBucket`

### 3️⃣ Install AWS CLI

```bash
sudo apt install awscli -y
```

### 4️⃣ Configure AWS CLI

```bash
aws configure
```

---

## 📜 Main Backup Script (Production‑Style)

```bash
#!/usr/bin/env bash

# Exit immediately on error
set -e

# Cron has a minimal PATH
PATH=/usr/local/bin:/usr/bin:/bin

# -------- CONFIG --------
LOG="/home/nrupal/backup.log"
S3_BUCKET="s3://local-backup-1201"

SRC="$1"
DEST="$2"
# ------------------------

# -------- VALIDATION --------
if [[ -z "$SRC" || -z "$DEST" ]]; then
  echo "$(date) ERROR: Source or destination missing" >> "$LOG"
  exit 1
fi

if [[ ! -d "$SRC" ]]; then
  echo "$(date) ERROR: Source directory does not exist: $SRC" >> "$LOG"
  exit 1
fi

mkdir -p "$DEST"
# ----------------------------

TIMESTAMP=$(date '+%Y-%m-%d-%H-%M')
ZIP_FILE="$DEST/backup-$TIMESTAMP.zip"

echo "----------------------------------------" >> "$LOG"
echo "$(date) Starting backup" >> "$LOG"

# -------- ZIP BACKUP --------
/usr/bin/zip -r "$ZIP_FILE" "$SRC" >> "$LOG" 2>&1

# -------- S3 SYNC --------
/usr/local/bin/aws s3 sync "$DEST" "$S3_BUCKET" >> "$LOG" 2>&1

echo "$(date) Backup completed & synced to S3" >> "$LOG"
```

---

## 🧪 How to Run Manually

```bash
chmod +x backup-to-s3.sh
./backup-to-s3.sh /home/nrupal/data /home/nrupal/backups
```

---

## ⏰ Cron Job Setup

```bash
crontab -e
```

Example (daily at 1 AM):

```bash
0 1 * * * /home/nrupal/practice/backup-to-s3.sh /home/nrupal/data /home/nrupal/backups
```

---

## 🧩 Easy Version (Beginner‑Friendly)

> Same idea, minimal logic — good for first‑time learners

```bash
#!/bin/bash

SRC="/home/nrupal/data"
DEST="/home/nrupal/backups"
BUCKET="s3://local-backup-1201"

mkdir -p "$DEST"

zip -r "$DEST/backup.zip" "$SRC"
aws s3 cp "$DEST/backup.zip" "$BUCKET"
```

---

## 🆚 Easy vs Production Script

| Easy Script     | Production Script |
| --------------- | ----------------- |
| Hardcoded paths | Accepts arguments |
| No logs         | Full logging      |
| No validation   | Safety checks     |
| Manual use      | Cron‑ready        |

---

## 📌 Why This Matters

Backups are **boring until they fail**.
This project helped me understand how real systems:

* Fail safely
* Log everything
* Run unattended

If you're learning DevOps, **start here**.

---

## 🤝 Contribution

If you're learning like me:

* Fork it
* Break it
* Improve it
* Share what you learn

---

## ⭐ Final Note

> I'm still learning — this repo exists so others can learn *with me*, not from a "perfect" solution.

Happy scripting 🐧
