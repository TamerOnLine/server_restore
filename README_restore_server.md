# restore-server Script

This script restores the **entire server** from a full backup archive.  
It rolls back all major system directories, website files, configurations, users, services, and cron jobs.

⚠️ **Warning:** This operation is destructive. Only run it if you want to completely restore your server to a previous state.

---

## Features
- Automatically detects the newest backup file.
- Restores:
  - `/etc`
  - `/etc/nginx`
  - `/etc/webapp`
  - `/etc/systemd/system`
  - `/var/www` (all websites)
  - `/home/tamer`
  - Cron jobs under `/var/spool/cron/crontabs`
- Reloads systemd and restarts Nginx safely.
- Fixes permissions for `/var/www`.

---

## Usage

### Restore using the latest backup:

```bash
sudo restore-server
```

### Restore using a specific backup file:

```bash
sudo restore-server /var/backups/server/server_2025-12-04_01-20-33.tar.gz
```

---

## Installation

Create the script:

```bash
sudo nano /usr/local/bin/restore-server
```

Paste the script content, then save and exit.

Make it executable:

```bash
sudo chmod +x /usr/local/bin/restore-server
```

---

## Confirmation Safety

To avoid accidental server overwrites, the script requires:

```
[type YES to continue]:
```

Only when you type **YES** will the restore begin.

---

## Notes

After restoring the full server, it is recommended to reboot:

```bash
sudo reboot
```

Nginx and system services may also need verification:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## Disclaimer
This script assumes that the backup archive was created using the matching backup system.  
Restoring an incompatible archive may overwrite important system files.

