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

## 5. How It Works Under the Hood

This script is where every earlier concept converges — read it as "the module, applied":

- **Validate-before-act (safety, [script-permissions](script-permissions.md)).** `SOURCE_DIR="${1:-}"` with `set -euo pipefail` and explicit `[ -d "$SOURCE_DIR" ]` checks means the script **cannot** proceed on an empty or wrong path — the guard that prevents `tar`/`rm` from acting on `/` or nothing.
- **`tar -C` for clean paths (archiving, [file-compression](../03-files-and-directories/file-compression.md)).** `tar -czf ARCHIVE -C "$(dirname SRC)" "$BASE"` changes into the *parent* directory first, so the archive stores `site/...` (relative) instead of `/var/www/site/...` (absolute). This matters at *restore* time: a relative archive extracts cleanly anywhere, while an absolute-path archive can clobber system paths.
- **Timestamped names + retention = a rotation policy.** `date +%F_%H%M` makes each run a unique file, so backups accumulate rather than overwrite. The prune step is the clever bit: `ls -1t ...*.tar.gz | tail -n +$((RETENTION+1))` lists newest-first, then `tail -n +N` selects everything *after* the first N — i.e., exactly the old ones to delete. The glob is **scoped** to `${BASE}-*.tar.gz`, so it can never match (and delete) anything but this backup's own files.
- **Verify success, don't assume it (troubleshooting mindset, Module 09).** `[ -s "$ARCHIVE" ]` confirms the file exists *and* is non-empty before pruning — so a failed `tar` (e.g., disk full mid-write) doesn't cause the script to delete good old backups while keeping a broken new one.

So the script is `set -euo pipefail` + input validation + `tar -C` + timestamped naming + verified-then-scoped prune — each line tracing to a principle from Modules 03, 08, 09, and 10.

## 6. Diagram

```mermaid
flowchart LR
    Src[Source dir] --> Validate[Validate inputs - safety]
    Validate --> Tar[tar -czf -C parent: clean paths]
    Tar --> Verify["Verify [ -s archive ]"]
    Verify --> Prune[Keep newest N, delete older - scoped glob]
    Prune --> Log[Log result]
```

## 7. Real-World Examples

**1. The everyday case.** Nightly, cron runs `backup.sh /var/www/site /backups`. Each morning there's a fresh `site-2026-07-02_0200.tar.gz`, and only the last 7 are kept — protecting against bad deploys or accidental deletes.

**2. Running it and seeing rotation work:**

```
$ ./backup.sh /etc /tmp/backups 3
Creating backup: /tmp/backups/etc-2026-07-02_1000.tar.gz
Backup OK: 2.1M
Pruning old backups (keeping 3)...
Removing old backup: /tmp/backups/etc-2026-06-29_1000.tar.gz
Backup complete.
$ ls -1t /tmp/backups
etc-2026-07-02_1000.tar.gz
etc-2026-07-01_1000.tar.gz
etc-2026-06-30_1000.tar.gz          # exactly 3 kept; older ones pruned
```

Retention held the newest 3 and removed the rest — the `tail -n +N` logic from Section 5.

**3. War story — the backup that quietly stopped working.** A team's backup ran nightly for a year, then the restore during an incident failed: the latest archive was **0 bytes**. The disk had filled weeks earlier, `tar` produced an empty file, but the *old* script had no verification — it happily pruned good backups and kept the broken one each night. This exact script's `[ -s "$ARCHIVE" ]` check (Section 5) would have *aborted before pruning*, preserving the last good backup and surfacing the failure. Lesson: an unverified, untested backup is not a backup — always verify, and periodically test a restore.

## 8. Worked Walkthrough

Run the full cycle and confirm each safety property:

```
$ chmod +x backup.sh
$ ./backup.sh            # no args → validation refuses (safety)
Usage: ./backup.sh <source-dir> <backup-dir> [retention]
$ ./backup.sh /etc /tmp/backups 2
Creating backup: /tmp/backups/etc-2026-07-02_1005.tar.gz
Backup OK: 2.1M
Backup complete.
$ ./backup.sh /etc /tmp/backups 2 ; ./backup.sh /etc /tmp/backups 2   # run twice more
$ ls -1t /tmp/backups | wc -l
2                        # retention=2 enforced — only newest 2 remain
$ tar -tzf "$(ls -1t /tmp/backups/*.tar.gz | head -1)" | head -3   # inspect newest
etc/
etc/hostname
etc/hosts               # relative paths (tar -C worked) → safe to restore anywhere
```

You saw validation refuse missing args, retention cap the count, and `tar -tzf` confirm clean relative paths — the design goals from Section 5.

## 9. Commands (the script)

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
TIMESTAMP="$(date +%F_%H%M)"      # e.g. 2026-07-02_0200
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

Sample output (dummy values, for reference):

```text
$ ./backup.sh /etc /tmp/backups 5
Creating backup: /tmp/backups/etc-2026-07-02_0200.tar.gz
Backup OK: 2.1M
Pruning old backups (keeping 5)...
Backup complete.

$ ls -lh /tmp/backups
-rw-r--r-- 1 alice alice 2.1M Jul  2 02:00 etc-2026-07-02_0200.tar.gz
```

## 10. Command Explanation (line by line, key parts)

- `set -euo pipefail` → abort on errors/unset vars (prevents acting on blank paths).
- `"${1:-}"` → read argument with empty default; then validated.
- `tar -czf ARCHIVE -C parent BASE` → `-C` changes into the parent so the archive stores a clean relative path (safe restore — Section 5).
- `[ -s "$ARCHIVE" ]` → true if the file exists **and** is non-empty (verifies success before pruning).
- `ls -1t ... | tail -n +N` → list newest-first, then select everything **after** the first N (the ones to delete).
- `rm -f -- "$old"` → `--` stops filenames being treated as options; the scoped glob only prunes matched backups, never the source.

## 11. In Production (DevOps Context)

- **Scheduled via cron/systemd timers** (Module 11) for hands-off nightly backups with rotation.
- **3-2-1 rule:** real backups live on a **different disk/host** (and offsite/object storage like S3) — a backup on the same disk dies with it.
- **Verify + test restores:** production backup jobs alert on failure and periodically test restores (the war story) — an untested backup is a liability.
- **This pattern generalizes:** database dumps (`pg_dump | gzip`), config snapshots, and artifact archiving all reuse validate→archive→verify→retain.

## 12. Practice Tasks

1. Run the script against `/etc` into `/tmp/backups`.
2. Run it several times; confirm timestamped files accumulate.
3. Set retention to 2 and verify older ones are pruned (watch the prune output).
4. Inspect an archive with `tar -tzf <file>` and confirm relative paths.

## 13. Common Mistakes

- Not validating inputs → `tar`/`rm` on an empty path (safety, [script-permissions](script-permissions.md)).
- Forgetting `-C`, so archives store full absolute paths (messy/unsafe restores).
- Skipping the `[ -s ]` verify, so a failed backup prunes good ones (the war story).
- Pruning with a glob that's too broad (this script scopes to `${BASE}-*.tar.gz`).

## 14. Troubleshooting

- **"No space left"** → backup dir disk is full (Module 08); change destination or free space (the empty-archive cause).
- **Permission denied reading source** → run with appropriate rights (maybe `sudo`).
- **Nothing pruned** → fewer backups than retention, or the glob didn't match; check naming.
- **Restore dumps files everywhere** → the archive had absolute paths; ensure `-C` is used.

## 15. Best Practices

- Store backups on a **different disk/host** than the source; follow 3-2-1.
- Verify archives (`[ -s ]`) and monitor backup success/failure.
- Schedule with cron (Module 11); log output to a file.
- Test restores periodically — an untested backup isn't a backup.

## 16. Connects To

- **Prev:** [Script Permissions & Safety](script-permissions.md). **Next:** [Log Cleanup Script Example](log-cleanup-script-example.md).
- **Concepts it applies:** [File Compression](../03-files-and-directories/file-compression.md), [Disk Full Troubleshooting](../08-storage-and-disk-management/disk-full-troubleshooting.md), safety from [Script Permissions](script-permissions.md).
- **Schedule it:** [Scheduled Backup Example](../11-automation-and-cron/scheduled-backup-example.md), [crontab Basics](../11-automation-and-cron/crontab-basics.md).

## 17. Quick Recap

- Validate → timestamped `tar.gz` (with `-C`) → **verify (`[ -s ]`)** → scoped prune → log.
- `-C` gives clean relative paths for safe restores; verify *before* pruning so a broken backup can't evict good ones.
- Schedule it, store it off-host, and test restores.

## 18. References

- `man tar`, `man date`, `man find`
- [Module 11 scheduled backup](../11-automation-and-cron/scheduled-backup-example.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Script Permissions & Safety](script-permissions.md) | ⬆️ Module: [Module 10 — Shell Scripting](README.md) | ➡️ Next: [Log Cleanup Script Example](log-cleanup-script-example.md) |
