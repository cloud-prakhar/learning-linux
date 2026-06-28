# Log Cleanup Script Example

## 1. What Is This?

A safe **log-cleanup script** that compresses or deletes log files older than a chosen number of days in a specific directory.

## 2. Why Is This Needed?

Logs fill disks (Module 08). A scheduled cleanup script keeps `/var/log` (or an app's log dir) under control automatically — without risking the wrong files.

## 3. Simple Layman Explanation

It's a **tidy-up robot** for one drawer of logs: it zips up papers older than a week and shreds anything older than a month — and it only ever touches the drawer you point it at.

## 4. Technical Explanation

The script:
1. Requires an explicit log directory (no default → can't run on the wrong place).
2. Validates the directory exists.
3. Compresses `.log` files older than `COMPRESS_DAYS`.
4. Deletes `.gz` files older than `DELETE_DAYS`.
5. Uses `find ... -print` so you can see exactly what it touched.

## 5. Real-World Example

Cron runs `cleanup-logs.sh /var/log/myapp 7 30` daily: logs older than 7 days get gzipped, archives older than 30 days get removed. The app's disk usage stops growing without manual effort.

## 6. Diagram

```mermaid
flowchart LR
    Dir[Log dir] --> Validate[Validate path]
    Validate --> Compress[gzip .log older than 7d]
    Compress --> Delete[delete .gz older than 30d]
    Delete --> Report[report actions]
```

## 7. Commands (the script)

Save as `cleanup-logs.sh`:

```bash
#!/bin/bash
# cleanup-logs.sh - compress old logs and delete very old archives in ONE directory
set -euo pipefail

LOG_DIR="${1:-}"                  # directory to clean (required, no default)
COMPRESS_DAYS="${2:-7}"           # gzip .log files older than this
DELETE_DAYS="${3:-30}"            # delete .gz files older than this

# --- Safety: require and validate the directory ---
if [ -z "$LOG_DIR" ]; then
    echo "Usage: $0 <log-dir> [compress_days] [delete_days]" >&2
    exit 1
fi
if [ ! -d "$LOG_DIR" ]; then
    echo "Error: '$LOG_DIR' is not a directory" >&2
    exit 1
fi

echo "Cleaning logs in: $LOG_DIR"
echo "Compress .log older than ${COMPRESS_DAYS}d, delete .gz older than ${DELETE_DAYS}d"

# --- Compress old, uncompressed logs ---
# -type f: files only | -name '*.log': only logs | -mtime +N: modified > N days ago
find "$LOG_DIR" -type f -name '*.log' -mtime +"$COMPRESS_DAYS" -print -exec gzip {} \;

# --- Delete archives older than DELETE_DAYS ---
find "$LOG_DIR" -type f -name '*.gz' -mtime +"$DELETE_DAYS" -print -delete

echo "Log cleanup complete."
```

Run (test safely first):

```bash
chmod +x cleanup-logs.sh
mkdir -p /tmp/logtest && touch -d '10 days ago' /tmp/logtest/old.log
./cleanup-logs.sh /tmp/logtest 7 30
ls -l /tmp/logtest          # old.log should now be old.log.gz
```

## 8. Command Explanation (key parts)

- `LOG_DIR="${1:-}"` with validation → the script **cannot** run without a valid directory (prevents accidents).
- `find ... -name '*.log' -mtime +7` → matches `.log` files modified **more than** 7 days ago.
- `-exec gzip {} \;` → compresses each match; `{}` is the filename, `\;` ends the command.
- `-name '*.gz' -mtime +30 -delete` → deletes only old gzip archives.
- `-print` → echoes every file acted on, so you have a record.

## 9. Practice Tasks

1. Create test logs with `touch -d 'N days ago'` and run the script.
2. Change `COMPRESS_DAYS`/`DELETE_DAYS` and observe the effect.
3. Add `-mtime +"$DELETE_DAYS"` dry-run by replacing `-delete` with `-print` first.
4. Schedule it daily (Module 11) once you trust it.

## 10. Common Mistakes

- Running on `/var/log` directly without testing on a temp dir first.
- Forgetting quotes around `"$LOG_DIR"` (breaks on spaces).
- Deleting active logs a process holds open (use truncate for those — Module 08).

## 11. Troubleshooting

- **Nothing happened** → no files matched the age/name; verify with `-print` and check `mtime`.
- **Permission denied** → the log dir needs higher privileges; run with appropriate rights.
- **Deleted too much** → you pointed it at the wrong dir; always test on `/tmp` first.

## 12. Best Practices

- Prefer **logrotate** for system logs; use this script for app dirs logrotate doesn't manage.
- Always dry-run (`-print` instead of `-delete`) before trusting it.
- Scope tightly by directory and file pattern.
- Don't delete logs an audit/compliance process needs.

## 13. Quick Recap

- Validate the dir → gzip old `.log` → delete old `.gz` → report.
- `find -mtime +N` selects by age; test with `-print` before `-delete`.
- For system logs, logrotate is preferred.

## 14. References

- `man find`, `man gzip`, `man touch`
- [Module 08 log cleanup](../08-storage-and-disk-management/log-cleanup-basics.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Backup Script Example](backup-script-example.md) | ⬆️ Module: [Module 10 — Shell Scripting](README.md) | ➡️ Next: [Module 11 — Automation & Cron](../11-automation-and-cron/README.md) |
