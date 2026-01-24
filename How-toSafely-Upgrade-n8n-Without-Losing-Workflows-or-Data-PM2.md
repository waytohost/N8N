# How to Safely Upgrade n8n (User / Local Install with PM2)

> **Applies to:** User-level (local) n8n installations managed by **PM2** on cPanel servers, VPS, and dedicated Linux servers

---

## 📌 Overview

This guide documents the **exact upgrade process used on this server**, where **n8n is installed under a user (not globally)** and managed by **PM2**.

In this setup:

* n8n is installed **locally inside the project directory**
* `package.json` controls the n8n version
* **`-g` (global install) is NOT used**
* Workflows and credentials are stored separately in the `.n8n` directory

---

## 📂 Where n8n Stores Data

All important n8n data is stored in the `.n8n` directory:

```
/root/.n8n
```

This includes:

* Workflows
* Credentials
* Execution history
* Encryption key and settings

✅ As long as this directory is preserved, upgrading n8n will **not** delete data.

---

## 💾 Backups Taken Before Upgrade

### 1️⃣ Backup n8n Data

```
cd /root
 tar -czvf .n8n.tar.gz .n8n
```

Additional safety backup:

```
cp -r ~/.n8n ~/.n8n-backup-$(date +%Y%m%d)
```

---

### 2️⃣ Backup Automation / Website Directory

```
cd /home2/yzzyconnect/public_html/automation.yzzyconnect.com
 tar -czvf automation.yzzyconnect.com.tar.gz *
```

---

## 🚀 Step-by-Step Upgrade Process (Actual Steps Used)

### Step 1: Check and Stop n8n

```
pm2 status
pm2 stop n8n
```

---

### Step 2: Backup `package.json`

```
cp -prf package.json package.json_bk
```

---

### Step 3: Edit `package.json`

```
vim package.json
```

Edit the dependency version:

```json
{
  "dependencies": {
    "n8n": "<desired-version>"
  }
}
```

> Example:
>
> ```json
> "n8n": "1.28.0"
> ```

---

### Step 4: Upgrade n8n (Local Install – NO `-g`)

From the project directory:

```
npm install n8n --unsafe-perm=true
```

Or install a specific version:

```
npm install n8n@1.28.0 --unsafe-perm=true
```

⚠️ Do **NOT** use:

```
npm install -g n8n
npm update n8n
```

---

### Step 5: Restart n8n

```
pm2 restart n8n
pm2 status
```

---

### Step 6: Verify Version

```
npx n8n --version
```

---

## 🔙 Rollback / Restore Guide

### Scenario 1: n8n Starts but Data Is Missing

```
pm2 stop n8n
mv ~/.n8n ~/.n8n-broken-$(date +%Y%m%d)
cp -r ~/.n8n-backup-YYYYMMDD ~/.n8n
pm2 restart n8n
```

---

### Scenario 2: n8n Fails to Start After Upgrade

```
pm2 logs n8n
pm2 stop n8n
npm install n8n@<previous-version> --unsafe-perm=true
pm2 restart n8n
```

---

### Scenario 3: Full Restore (Last Resort)

```
pm2 stop n8n
rm -rf node_modules/n8n
npm install n8n@<previous-version> --unsafe-perm=true
cp -r ~/.n8n-backup-YYYYMMDD ~/.n8n
pm2 restart n8n
```

---

## 📋 Best Practices (Local Install)

* Always stop n8n before upgrading
* Always back up `.n8n`
* Do **not** use `-g` for user installs
* Control versions via `package.json`
* Use `pm2 restart`, not `pm2 start`
* Keep at least one working backup

---

## 🧠 Final Notes

This page is **ready to copy–paste directly into a GitHub Wiki**.

It reflects the **exact upgrade method used on this server**, ensuring safe upgrades and easy rollback for **user-installed n8n running under PM2**.
