# Server and Site Restore Scripts

## Overview

This repository contains two Bash scripts for restoring backups on a Linux server:

- `restore-server` – performs a **full server restore** from a tarball backup created under `/var/backups/server`. fileciteturn0file0  
- `restore-site` – restores a **single site** (code + config) from a per-site backup under `/var/backups/sites/<site_name>`. fileciteturn0file1  

Both scripts are designed to be run **manually as `root` (or via `sudo`)** and assume a conventional web hosting layout under `/var/www` with Nginx as the web server.

> ⚠️ Restoring from backups will overwrite existing files. Always ensure you know exactly which backup you are restoring and that you have current backups before proceeding.

---

## Requirements

### Server Environment

- Linux server (Debian/Ubuntu or similar)
- `bash` shell
- `tar` command-line tool
- `sudo` access or direct `root` login
- Optional but expected:
  - `systemd` (for `systemctl` reload/restart calls) fileciteturn0file0  
  - `nginx` (web server) fileciteturn0file0  

### Expected Directory Layout

The scripts assume backups and sites are stored in the following locations:

- Full server backups:
  - `/var/backups/server/server_YYYY-MM-DD_HH-MM-SS.tar.gz` fileciteturn0file0  
- Per-site backups:
  - `/var/backups/sites/<site_name>/<site_name>_YYYY-MM-DD_HH-MM-SS.tar.gz` fileciteturn0file1  

Runtime paths that may be overwritten by restores include:

- `/etc`
- `/etc/nginx`
- `/etc/webapp`
- `/etc/systemd/system`
- `/var/www`
- `/home/tamer`
- `/var/spool/cron/crontabs` fileciteturn0file0  

The scripts also fix file ownership after restore using:

- User: `tamer`
- Group: `www-data`

If your environment uses different usernames or groups, you should **adjust the scripts accordingly** before use.

> **Note:** The presence of user `tamer` and group `www-data` is assumed based on the scripts. If your server does not use these, update the `chown` lines or create equivalent users/groups.

---

## Local Installation (Development / Review)

These scripts can be reviewed or edited locally on your workstation before deployment to the server.

1. **Clone or download the project**

   ```bash
   # Example if this is in a Git repository
   git clone <your-repo-url> restore-tools
   cd restore-tools
   ```

   Or copy the two provided files into a directory:

   - `restore-server.sh`
   - `restore-site.sh`

2. **Make the scripts executable**

   ```bash
   chmod +x restore-server.sh restore-site.sh
   ```

3. **Optional: Run shellcheck (lint) locally**

   ```bash
   shellcheck restore-server.sh restore-site.sh
   ```

   This is optional but recommended for verifying shell script quality.

---

## Uploading the Project to the Server

You typically want these scripts available globally on your server, e.g. under `/usr/local/sbin` so you can call `restore-server` or `restore-site` directly.

### 1. Copy scripts from your local machine to the server

From your local machine:

```bash
# Adjust `user` and `server.example.com` to your SSH user and host
scp restore-server.sh restore-site.sh user@server.example.com:/tmp/
```

### 2. SSH into the server

```bash
ssh user@server.example.com
```

### 3. Move scripts into a system path and rename

```bash
sudo mv /tmp/restore-server.sh /usr/local/sbin/restore-server
sudo mv /tmp/restore-site.sh /usr/local/sbin/restore-site
```

### 4. Make them executable

```bash
sudo chmod +x /usr/local/sbin/restore-server /usr/local/sbin/restore-site
```

Now you can invoke them as:

```bash
sudo restore-server
sudo restore-site <site_name>
```

---

## Installing Dependencies on the Server

On a Debian/Ubuntu-based system, ensure the following packages are installed:

```bash
sudo apt update
sudo apt install -y bash tar nginx
```

`systemd` is usually present by default on modern distributions. If `nginx` is not installed, the scripts will still run, but the automatic restart of Nginx will fail silently (it is wrapped in a safe check). fileciteturn0file0  

If you use a different web server or init system, you may want to adjust or remove the `systemctl` and `nginx` calls in the scripts.

---

## Usage

### 1. Full Server Restore (`restore-server`)

Script: `restore-server` (original file: `restore-server.sh`) fileciteturn0file0  

**Description:**  
Restores a **full server configuration and data** from a tarball backup. This is a destructive operation intended for complete rollbacks or disaster recovery.

#### Usage

```bash
# Basic usage – restore from the newest backup in /var/backups/server
sudo restore-server

# Restore from a specific backup file
sudo restore-server /var/backups/server/server_2025-12-04_01-20-33.tar.gz
```

#### What the script does

1. Ensures it is run as `root` (or via `sudo`).  
2. If no backup path is provided, automatically selects the **newest** `server_*.tar.gz` from `/var/backups/server`.  
3. Displays the backup file being used and a list of paths that may be overwritten. fileciteturn0file0  
4. Asks for confirmation with a strong safeguard:

   ```text
   Are you absolutely sure you want to continue? [type YES to continue]:
   ```

   Only if you type `YES` (uppercase) does the restore proceed.

5. Extracts the backup archive into system root `/`:

   ```bash
   tar -xzf "$BACKUP_FILE" -C /
   ```

6. Reloads systemd and restarts Nginx (if available): fileciteturn0file0  

   ```bash
   systemctl daemon-reload
   systemctl restart nginx
   ```

7. Fixes permissions for the website directories under `/var/www`:

   ```bash
   chown -R tamer:www-data /var/www
   ```

8. Recommends rebooting the server after restore.

#### Recommended procedure

1. Ensure you have a **current backup** as well as the backup you intend to restore.  
2. Verify disk space is sufficient for restore.  
3. Run:

   ```bash
   sudo restore-server
   ```

4. Type `YES` when you are absolutely sure.  
5. After completion, run a quick check:

   ```bash
   sudo systemctl status nginx
   ```

6. Reboot the server, as recommended:

   ```bash
   sudo reboot
   ```

> ⚠️ This script overwrites core directories like `/etc`, `/var/www`, and user directories. Treat it as a full system rollback.

---

### 2. Single Site Restore (`restore-site`)

Script: `restore-site` (original file: `restore-site.sh`) fileciteturn0file1  

**Description:**  
Restores a **single website/application** from its site-specific backup. This is less invasive than a full server restore and targets only one site and its related configuration.

#### Usage

```bash
# Basic usage – restore the newest backup for a site
sudo restore-site <site_name>

# Example
sudo restore-site mystrotamer

# Restore from a specific backup file
sudo restore-site mystrotamer /var/backups/sites/mystrotamer/mystrotamer_2025-12-04_01-02-37.tar.gz
```

#### What the script does

1. Ensures it is run as `root` (or via `sudo`). fileciteturn0file1  
2. Requires a `site_name` argument; prints usage and exits if missing.  
3. If no backup file is provided, automatically selects the **newest** backup file matching:

   ```text
   /var/backups/sites/<site_name>/<site_name>_*.tar.gz
   ```

4. Confirms that a valid backup file exists; otherwise prints a clear error and exits. fileciteturn0file1  
5. Shows which site and backup file will be restored and warns about overwriting:

   - `/var/www/<site_name>`
   - `/etc/webapp/<site_name>.env` (if present in backup)
   - `/etc/nginx/sites-available` (if configuration is included) fileciteturn0file1  

6. Asks for confirmation:

   ```text
   Are you sure you want to continue? [y/N]:
   ```

7. If confirmed, extracts the backup into `/` with:

   ```bash
   tar -xzf "$BACKUP_FILE" -C /
   ```

8. Fixes permissions for the project folder:

   ```bash
   chown -R tamer:www-data "/var/www/$SITE_NAME"
   ```

9. Prints a reminder to test and reload Nginx if configs were restored:

   ```bash
   sudo nginx -t && sudo systemctl reload nginx
   ```

#### Recommended procedure

1. Identify the site name (directory under `/var/www` and backups under `/var/backups/sites`).  
2. List available backups:

   ```bash
   ls -lh /var/backups/sites/<site_name>/
   ```

3. Restore the newest backup:

   ```bash
   sudo restore-site <site_name>
   ```

4. After successful restore, verify the site in a browser and check Nginx:

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

## Production Considerations

- **Access control**: These scripts should only be executable by trusted administrators. Restrict SSH access and ensure `/usr/local/sbin` is not writable by non-admin users.
- **Backups frequency**: Ensure your backup routine runs reliably and that you periodically test restoring to a staging server to validate integrity.
- **User and group**: If you change `tamer` or `www-data` to other names, ensure directory ownership across `/var/www` and service configs is consistent.

---

## Optional systemd Service Setup

These scripts are **intended for manual execution** because they can overwrite large parts of the system and individual sites. It is generally **not recommended** to run them automatically via `systemd` or cron.

However, if you still want a controlled wrapper (for example, to allow a supervised one-shot restore), you could create a **one-shot systemd unit** that an administrator must explicitly trigger.

> ⚠️ Use with extreme caution. Automating restores can lead to unexpected data loss.

Example (for illustration only) – `/etc/systemd/system/manual-restore-site@.service`:

```ini
[Unit]
Description=Manual restore of site %i from latest backup
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/restore-site %i
User=root

[Install]
WantedBy=multi-user.target
```

Reload systemd and run manually:

```bash
sudo systemctl daemon-reload
sudo systemctl start manual-restore-site@mystrotamer.service
```

Again, this pattern is **optional and not recommended** for regular use.

---

## Optional Nginx Reverse Proxy Example

These scripts assume Nginx is installed and that site configurations live under `/etc/nginx/sites-available` (and likely symlinked in `/etc/nginx/sites-enabled`). fileciteturn0file1  

A minimal reverse proxy configuration for a site might look like this (for context; not directly enforced by the scripts):

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/example.com/public;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

If your backups include Nginx configs, restoring a site may overwrite the file at `/etc/nginx/sites-available/example.com`. Always test Nginx before reloading:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## Troubleshooting

### 1. "❌ Please run this script as root" error

**Cause:** Script is being executed without root privileges. fileciteturn0file0turn0file1  

**Fix:** Run with `sudo`:

```bash
sudo restore-server
sudo restore-site <site_name>
```

---

### 2. "❌ Server backup file not found."

**Cause:** No matching backup file in `/var/backups/server` or the file path you provided does not exist. fileciteturn0file0  

**Fixes:**

- Verify backups:

  ```bash
  ls -lh /var/backups/server/
  ```

- If you have a custom path, pass it explicitly:

  ```bash
  sudo restore-server /path/to/your/server_backup.tar.gz
  ```

---

### 3. "❌ Backup file not found for site '<site_name>'."

**Cause:** No backup files found under `/var/backups/sites/<site_name>` or wrong site name. fileciteturn0file1  

**Fixes:**

- Double-check site name and directory:

  ```bash
  ls -lh /var/backups/sites/
  ls -lh /var/backups/sites/<site_name>/
  ```

- If needed, provide the exact backup path:

  ```bash
  sudo restore-site <site_name> /var/backups/sites/<site_name>/<site_name>_YYYY-MM-DD_HH-MM-SS.tar.gz
  ```

---

### 4. Nginx fails to restart or reload

**Symptoms:** After restore, `nginx -t` fails, or `systemctl restart nginx` returns an error.

**Fixes:**

1. Test the configuration:

   ```bash
   sudo nginx -t
   ```

2. Review recently restored config files in `/etc/nginx/sites-available` and `/etc/nginx/nginx.conf`.  
3. Fix syntax errors or invalid references, then run:

   ```bash
   sudo systemctl reload nginx
   ```

If Nginx was not installed, you can either install it or remove/ignore Nginx-specific lines in the scripts.

---

### 5. Permissions issues after restore

**Symptoms:** Web application cannot read/write files, 403 errors, or internal errors referencing filesystem permissions.

**Cause:** File ownership may not match your actual runtime user/group.

**Fixes:**

- Confirm your web user and group (e.g. `www-data`, `nginx`, etc.).  
- Adjust the `chown` commands in the scripts or run a one-time correction, for example:

  ```bash
  sudo chown -R tamer:www-data /var/www
  ```

  or replace with your actual user/group:

  ```bash
  sudo chown -R myuser:mygroup /var/www
  ```

---

## Notes and Assumptions

- User `tamer` and group `www-data` are assumed from the original scripts; you may need to change these for your environment.
- Backups are assumed to be **valid tar.gz archives** that were created with paths relative to `/` or the appropriate root.
- The documentation assumes a Debian/Ubuntu-type system; for other distributions, package manager commands and service management commands may differ.

If you adapt these scripts for another layout or user model, update this README accordingly so future administrators understand how restores work on your infrastructure.
