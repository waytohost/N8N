```markdown
# How to Safely Upgrade n8n Without Losing Workflows or Data (PM2)

> **Applies to:** PM2-based n8n installations on cPanel servers, VPS, and dedicated Linux servers

---

## 📌 Overview

Upgrading **n8n** is necessary to receive new features, bug fixes, security patches, and performance improvements. When done correctly, upgrading **does not delete workflows, credentials, or execution data**.

This guide explains:

- Where n8n stores its data  
- How to safely back it up  
- How to upgrade n8n when running under **PM2**  
- How to roll back safely if something goes wrong  

---

## 📂 Where n8n Stores Your Data

All important n8n data is stored in the **`.n8n` directory**, including:

- Workflows  
- Credentials  
- Execution history  
- Encryption key and configuration  

**Default location:**

```

/home/your-username/.n8n

```

✅ As long as this directory is not deleted or overwritten, your data will remain intact after upgrading n8n.

---

## 💾 Recommended: Back Up n8n Data

Always back up the `.n8n` directory before upgrading.

```

cp -r ~/.n8n ~/.n8n-backup-$(date +%Y%m%d)

```

---

## 🌐 Optional: Backup Website / Document Root (cPanel or Any Server)

If your server hosts websites, you may also want a quick document root backup.

```

cd /home/username/public_html
tar -czvf domain-backup-$(date +%Y%m%d).tar.gz *

```

---

## 🚀 Step-by-Step: Upgrade n8n

### Step 1: Stop n8n

**Using PM2 (recommended):**

```

pm2 stop n8n
pm2 status

```

**Using systemd:**

```

sudo systemctl stop n8n

```

---

### Step 2: Upgrade n8n via npm

✅ **Recommended method:**

```

sudo npm install -g n8n --unsafe-perm=true

```

⚠️ **Not recommended:**

```

sudo npm update -g n8n

```

---

## 🛠️ Troubleshooting Common Errors

### ENOTEMPTY Error

```

npm ERR! ENOTEMPTY: directory not empty, rename
'/usr/lib/node_modules/n8n' -> '/usr/lib/node_modules/.n8n-xxxx'

```

**Fix:**

```

sudo rm -rf /usr/lib/node_modules/.n8n-*
sudo npm install -g n8n --unsafe-perm=true

```

**Permission fix (if needed):**

```

sudo chown -R $(whoami) /usr/lib/node_modules

```

**Last resort:**

```

sudo npm install -g n8n --unsafe-perm=true --force

```

---

## ✅ Verify Upgrade

```

n8n --version

```

---

## 🔄 Restart n8n

**PM2:**

```

pm2 restart n8n
pm2 status

```

**systemd:**

```

sudo systemctl start n8n
sudo systemctl status n8n

```

---

## 🔁 Enable Auto-Start on Boot

**PM2:**

```

pm2 startup
pm2 save

```

**systemd:**

```

sudo systemctl enable n8n

```

---

## 🔙 Rollback / Restore Guide (If Upgrade Fails)

### Scenario 1: n8n Starts but Data Is Missing

```

pm2 stop n8n
mv ~/.n8n ~/.n8n-broken-$(date +%Y%m%d)
cp -r ~/.n8n-backup-YYYYMMDD ~/.n8n
pm2 restart n8n

```

---

### Scenario 2: n8n Fails to Start

```

pm2 logs n8n
sudo npm install -g n8n@<previous-version> --unsafe-perm=true
cp -r ~/.n8n-backup-YYYYMMDD ~/.n8n
pm2 restart n8n

```

Example:

```

sudo npm install -g n8n@1.28.0 --unsafe-perm=true

```

---

### Scenario 3: Full Restore (Last Resort)

```

pm2 stop n8n
sudo npm uninstall -g n8n
sudo npm install -g n8n --unsafe-perm=true
cp -r ~/.n8n-backup-YYYYMMDD ~/.n8n
pm2 restart n8n

```

---

## 📋 Best Practices Summary

- Stop n8n before upgrading  
- Always back up `.n8n`  
- Use `npm install -g n8n --unsafe-perm=true`  
- Keep at least one known-good backup  
- Upgrade during low-traffic hours  

---

## 🧠 Final Notes

This page is ready to **copy–paste directly into a GitHub Wiki**.

Following this guide ensures safe **upgrade, rollback, and recovery** of n8n on **cPanel servers, VPSs, and dedicated Linux servers** running n8n with PM2.
```
