# restore-site Script

This script restores a single FastAPI website from a backup archive on your server.

## Features
- Restore a specific site from `/var/www/<site>`
- Restore environment variables from `/etc/webapp/<site>.env`
- Restore Nginx configuration if included
- Automatically select the latest backup if none is specified
- Safe confirmation prompt before restoring

## Usage

Restore the newest backup:

```bash
sudo restore-site <site_name>
```

Restore using a specific backup file:

```bash
sudo restore-site <site_name> /var/backups/sites/<site>/<filename>.tar.gz
```

## Installation

Create the script:

```bash
sudo nano /usr/local/bin/restore-site
```

Paste the script content, then save.

Make it executable:

```bash
sudo chmod +x /usr/local/bin/restore-site
```

## Example

```bash
sudo restore-site mystrotamer
```

## Notes
- The script must be run as root.
- After restoring, you may need to reload Nginx:

```bash
sudo nginx -t && sudo systemctl reload nginx
```
