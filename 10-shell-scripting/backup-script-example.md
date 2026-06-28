# Backup Script Example

## 1. What Is This?

A complete, beginner-friendly **backup script** that archives a directory into a timestamped `.tar.gz` and keeps only the most recent backups.

## 2. Why Is This Needed?

Backups are essential and easy to forget. A script makes them consistent, timestamped, and schedulable (with cron, Module 11) — turning "I should back up" into "it backs up automatically."

## 3. Simple Layman Explanation

This script is an **automatic photocopier with a filing system**: it copies your folder into a dated, compressed package and throws out packages older than the few you want to keep.

## 4. Technical Explanation

The script:
1. Validates the source directory and backup destination.
2. Creates a timestamped `tar.gz` archive.
3. Verifies the archive was created.
4. Deletes backups beyond a retention count.
5. Logs each step. No destructive command runs on unvalidated input.

## 5. Real-World Example

Nightly, cron runs `backup.sh /var/www/site /backups`. Each morning there's a fresh `site-2026-06-28_0200.tar.gz`, and only the last 7 are kept — protecting against bad deploys or accidental deletes.

## 6. Diagram

```mermaid
flowchart LR
    Src[Source dir] --> Validate[Validate inputs]
    Validate --> Tar[Create timestamped tar.gz]
    Tar --> Verify[Verify archive]
    Verify --> Prune[Keep last N, delete older]
    Prune --> Log[Log result]
```

## 7. Commands (the script)

Save as `backup.sh`:

```bash
#!/bin/bash
# backup.sh - archive a directory into a timestamped tar.gz and prune old backups
set -euo pipefail

# --- Configuration / arguments ---
SOURCE_DIR="${1:-}"                 # directory to back up
BACKUP_DIR="${2:-}"                 # where to store backups
RETENTION="${3:-7}"                 # how many backups to keep (default 7)

# --- Validate inputs (safety first) ---
if [ -z "$SOURCE_DIR" ] || [ -z "$BACKUP_DIR" ]; then
    echo "Usage: $0 <source-dir> <backup-dir> [retention]" >&2
    exit 1
fi
if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: source '$SOURCE_DIR' is not a directory" >&2
    exit 1
fi

mkdir -p "$BACKUP_DIR"             # create destination if missing

# --- Build a timestamped archive name ---
TIMESTAMP="$(date +%F_%H%M)"      # e.g. 2026-06-28_0200
BASE="$(basename "$SOURCE_DIR")"  # last path component
ARCHIVE="$BACKUP_DIR/${BASE}-${TIMESTAMP}.tar.gz"

# --- Create the archive ---
echo "Creating backup: $ARCHIVE"
tar -czf "$ARCHIVE" -C "$(dirname "$SOURCE_DIR")" "$BASE"

# --- Verify it exists and is non-empty ---
if [ -s "$ARCHIVE" ]; then
    echo "Backup OK: $(du -h "$ARCHIVE" | cut -f1)"
else
    echo "Error: backup archive missing or empty" >&2
    exit 1
fi

# --- Prune old backups, keeping the newest $RETENTION ---
echo "Pruning old backups (keeping $RETENTION)..."
ls -1t "$BACKUP_DIR/${BASE}-"*.tar.gz 2>/dev/null \
    | tail -n +"$((RETENTION + 1))" \
    | while read -r old; do
        echo "Removing old backup: $old"
        rm -f -- "$old"
      done

echo "Backup complete."
```

Run:

```bash
chmod +x backup.sh
./backup.sh /etc /tmp/backups 5      # back up /etc, keep 5
ls -lh /tmp/backups
```

## 8. Command Explanation (line by line, key parts)

- `set -euo pipefail` → abort on errors/unset vars (prevents acting on blank paths).
- `"${1:-}"` → read argument with empty default; then validated.
- `tar -czf ARCHIVE -C parent BASE` → `-C` changes into the parent so the archive stores a clean relative path.
- `[ -s "$ARCHIVE" ]` → true if the file exists **and** is non-empty (verifies success).
- `ls -1t ... | tail -n +N` → list newest-first, then select everything **after** the first N (the ones to delete).
- `rm -f -- "$old"` → `--` stops filenames being treated as options; only prunes matched backups, never the source.

## 9. Practice Tasks

1. Run the script against `/etc` into `/tmp/backups`.
2. Run it several times; confirm timestamped files accumulate.
3. Set retention to 2 and verify older ones are pruned.
4. Inspect an archive with `tar -tzf <file>`.

## 10. Common Mistakes

- Not validating inputs → `tar`/`rm` on an empty path.
- Forgetting `-C`, so archives store full absolute paths.
- Pruning with a glob that's too broad (this script scopes to `${BASE}-*.tar.gz`).

## 11. Troubleshooting

- **"No space left"** → backup dir disk is full (Module 08); change destination or free space.
- **Permission denied reading source** → run with appropriate rights (maybe `sudo`).
- **Nothing pruned** → fewer backups than retention, or the glob didn't match; check naming.

## 12. Best Practices

- Store backups on a **different disk/host** than the source.
- Verify archives and monitor backup success.
- Schedule with cron (Module 11); log output to a file.
- Test restores periodically — an untested backup isn't a backup.

## 13. Quick Recap

- Validate → timestamped `tar.gz` → verify → prune → log.
- `-C` for clean paths; `[ -s ]` to verify; scoped prune glob.
- Schedule it and test restores.

## 14. References

- `man tar`, `man date`, `man find`
- [Module 11 scheduled backup](../11-automation-and-cron/scheduled-backup-example.md)
