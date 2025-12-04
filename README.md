# Restore System — Site & Full Server Recovery

## 📌 Overview
This document explains the complete **Restore System** used to recover:
- A single website  
- A specific backup version  
- The entire server  
- System-wide configurations  

It is designed to work with the backup structure created by:
- backup-site  
- backup-server  

---

# 📁 Restore Structure

Restore Type | Script | Example Usage
------------ | ------ | --------------
Restore single site | `restore-site` | `sudo restore-site mystrotamer`
Restore specific backup file | `restore-site` | `sudo restore-site mystrotamer /path/to/backup.tar.gz`
Restore full server | `restore-server` | `sudo restore-server`
Restore specific server version | `restore-server` | `sudo restore-server /path/to/server_backup.tar.gz`

Backups must be located under:

```
/var/backups/sites/<site_name>/
/var/backups/server/
```

---

# 🔄 1. Restore a Single Site

## ✔️ Restore the latest backup

```
sudo restore-site <site_name>
```

Example:

```
sudo restore-site mystrotamer
```

This will restore:
- `/var/www/<site>`
- `/etc/webapp/<site>.env`
- nginx config (if included)

---

## ✔️ Restore a specific backup file

```
sudo restore-site <site_name> /var/backups/sites/<site>/<backup>.tar.gz
```

Example:

```
sudo restore-site mystrotamer /var/backups/sites/mystrotamer/mystrotamer_2025-12-04_01-02.tar.gz
```

---

# 🖥️ 2. Restore the Entire Server

## ✔️ Restore the latest version

```
sudo restore-server
```

Restores:
- `/etc`
- `/etc/nginx`
- `/etc/webapp`
- `/etc/systemd/system`
- `/var/www`
- `/home/tamer`
- cron jobs

⚠️ **Danger:** This overwrites system directories.

---

## ✔️ Restore a specific server backup

```
sudo restore-server /var/backups/server/server_YYYY-MM-DD_HH-MM.tar.gz
```

Example:

```
sudo restore-server /var/backups/server/server_2025-12-04_01-20.tar.gz
```

---

# 🔧 3. Actions Required After Restore

Run:

```
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl daemon-reload
sudo reboot
```

---

# 📁 Backup Locations

### Site backups:
```
/var/backups/sites/<site_name>/
```

### Server backups:
```
/var/backups/server/
```

---

# 🛠️ Restore Scripts Location

```
/usr/local/bin/restore-site
/usr/local/bin/restore-server
```

Make them executable:

```
sudo chmod +x /usr/local/bin/restore-site
sudo chmod +x /usr/local/bin/restore-server
```

---

# 👨‍💻 Author
Created by **TamerOnLine**  
Production-grade restore system for Linux servers.

