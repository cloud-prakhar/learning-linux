# Log Cleanup Basics

## 1. What Is This?

Safely reducing the space consumed by log files — using **log rotation**, **journald limits**, and careful manual cleanup.

## 2. Why Is This Needed?

Logs grow forever if unmanaged and are the #1 cause of full disks. Clean them the right way to free space without losing important records or breaking the apps that write them.

## 3. Simple Layman Explanation

Logs are like a diary that never stops. Instead of throwing out the whole diary (risky), you archive old pages, keep recent ones, and set a maximum number of pages — automatically.

## 4. Technical Explanation

- **logrotate** rotates, compresses, and deletes old logs on a schedule (config in `/etc/logrotate.conf` and `/etc/logrotate.d/`).
- **systemd journald** stores binary logs; cap them with `--vacuum-size`/`--vacuum-time` or `SystemMaxUse` in `/etc/systemd/journald.conf`.
- For a **growing, in-use** log, use `truncate -s 0` (not `rm`), so the writing process keeps its file handle.

## 5. Real-World Example

`/var/log/app/app.log` is 20G because the app has no rotation. You truncate it to reclaim space now, then add a logrotate rule so it auto-rotates daily, keeps 7 days, and compresses — the problem never recurs.

## 6. Diagram

```mermaid
flowchart LR
    Log[app.log grows] --> Rotate[logrotate: rotate daily]
    Rotate --> Keep[keep 7, compress old]
    Keep --> Delete[delete beyond retention]
```

## 7. Commands

```bash
sudo du -ah /var/log | sort -h | tail        # biggest logs
sudo truncate -s 0 /var/log/app/app.log      # empty an active log safely
sudo journalctl --disk-usage                 # journal size
sudo journalctl --vacuum-size=200M           # cap journal to 200M
sudo journalctl --vacuum-time=7d             # keep only 7 days
sudo logrotate -d /etc/logrotate.conf        # dry-run (debug) rotation
sudo logrotate -f /etc/logrotate.d/myapp     # force a rotation now
```

Example logrotate rule (`/etc/logrotate.d/myapp`):

```
/var/log/app/*.log {
    daily              # rotate every day
    rotate 7           # keep 7 old versions
    compress           # gzip old logs
    missingok          # don't error if missing
    notifempty         # skip if empty
    copytruncate       # truncate the original (for apps that hold it open)
}
```

## 8. Command Explanation

- `truncate -s 0 file` → empties a log in place; safe while the app is writing.
- `journalctl --vacuum-size/--vacuum-time` → trims systemd's binary journal.
- `logrotate -d` → **dry run**; shows what *would* happen without changing anything.
- `logrotate -f` → forces rotation immediately (useful for testing your rule).
- `copytruncate` → copies then truncates the live file, so apps without reopen-on-rotate keep logging.

## 9. Practice Tasks

1. `sudo du -ah /var/log | sort -h | tail` — find your biggest logs.
2. `sudo journalctl --disk-usage`, then `sudo journalctl --vacuum-time=7d`.
3. Create a test log, fill it, then `truncate -s 0` it and watch the size drop.
4. Write a logrotate rule for a test path and run `logrotate -d` on it.

## 10. Common Mistakes

- `rm`-ing an active log instead of truncating → space not freed until the app restarts.
- Deleting logs an audit/compliance process needs.
- Forgetting `copytruncate` for apps that don't reopen their log after rotation.

## 11. Troubleshooting

- **Space not freed after deleting a log** → the app still holds it open; `truncate` instead, or restart the app.
- **logrotate not running** → check it's scheduled (cron/systemd timer) and the config path is correct.
- **App stopped logging after rotation** → it didn't reopen the file; use `copytruncate` or send it a reload signal.

## 12. Best Practices

- Automate with **logrotate**; never rely on manual cleanup.
- Cap journald size in `journald.conf` (`SystemMaxUse=`).
- Use `truncate` for active logs; test rotation with `-d` first.

## 13. Quick Recap

- Use logrotate (daily, keep N, compress) + journald limits.
- `truncate -s 0` for active logs; `rm` can fail to free space.
- Test with `logrotate -d` before relying on it.

## 14. References

- `man logrotate`, `man journalctl`, `man journald.conf`
- [disk-full-troubleshooting.md](./disk-full-troubleshooting.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Disk Full Troubleshooting](disk-full-troubleshooting.md) | ⬆️ Module: [Module 08 — Storage & Disk Management](README.md) | ➡️ Next: [Module 09 — Logs, Monitoring & Troubleshooting](../09-logs-monitoring-troubleshooting/README.md) |
