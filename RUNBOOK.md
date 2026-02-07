# 69Pool Stats Runbook

This runbook covers the `ckstats` web app on the stats droplet.

---

## Locations

- App: `/opt/ckstats`
- Env: `/opt/ckstats/.env`
- Logs sync target: `/var/lib/ckstats/logs`
- Systemd service: `/etc/systemd/system/ckstats.service`
- Reverse proxy: `/etc/caddy/Caddyfile`

---

## Service Control

Start:
```bash
sudo systemctl start ckstats
```

Stop:
```bash
sudo systemctl stop ckstats
```

Restart:
```bash
sudo systemctl restart ckstats
```

Status:
```bash
sudo systemctl status ckstats
```

Logs:
```bash
sudo journalctl -u ckstats -f
```

---

## Build and Deploy (Stats Droplet)

```bash
cd /opt/ckstats
git pull
pnpm install
pnpm build
sudo systemctl restart ckstats
```

---

## Environment Config

Edit:
```bash
sudo nano /opt/ckstats/.env
```

Typical values:
```
API_URL="/var/lib/ckstats/logs"
DB_HOST="127.0.0.1"
DB_PORT="5432"
DB_USER="ckstats"
DB_PASSWORD="REDACTED"
DB_NAME="ckstats"
```

After changes:
```bash
sudo systemctl restart ckstats
```

---

## Cron Jobs (ckstats)

Edit:
```bash
crontab -e
```

Expected entries (adjust pnpm path if needed):
```
*/1 * * * * cd /opt/ckstats && /root/.local/share/pnpm/pnpm seed
*/1 * * * * cd /opt/ckstats && /root/.local/share/pnpm/pnpm update-users
5 */2 * * * cd /opt/ckstats && /root/.local/share/pnpm/pnpm cleanup
```

List:
```bash
crontab -l
```

---

## Log Sync (Pool -> Stats)

Script:
```bash
/usr/local/bin/sync-ckpool-logs.sh
```

Manual run:
```bash
/usr/local/bin/sync-ckpool-logs.sh
```

Cron (every minute):
```
*/1 * * * * /usr/local/bin/sync-ckpool-logs.sh
```

Verify logs are present:
```bash
ls -la /var/lib/ckstats/logs
```

---

## Reverse Proxy (Caddy)

Config:
```bash
cat /etc/caddy/Caddyfile
```

Reload:
```bash
sudo systemctl reload caddy
```

Status:
```bash
sudo systemctl status caddy
```

---

## Troubleshooting

ckstats not responding:
1. `sudo systemctl status ckstats`
2. `sudo journalctl -u ckstats -f`
3. Confirm `pnpm` path in `ckstats.service`.

No stats data:
1. Check `sync-ckpool-logs.sh` cron.
2. Confirm logs in `/var/lib/ckstats/logs`.
3. Run `pnpm seed` manually.
