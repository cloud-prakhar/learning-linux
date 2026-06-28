# Disk Full Troubleshooting

## 1. What Is This?

A step-by-step method to diagnose and fix a **full disk** — one of the most common and disruptive Linux incidents.

## 2. Why Is This Needed?

When a disk hits 100%, apps can't write, databases crash, logins may fail, and deploys break. You need to free space safely and fast, without deleting something critical.

## 3. Simple Layman Explanation

Your shelf is overflowing. You: see which shelf is full (`df`), find the heaviest boxes (`du`), throw out clearly-junk boxes (old logs, caches), and stop it filling again (rotation/monitoring).

## 4. Technical Explanation

Method:
1. **Find the full filesystem** → `df -h`.
2. **Drill into big directories** → `du -sh /path/* | sort -h`.
3. **Identify safe-to-remove data** → old logs, temp files, caches, old packages.
4. **Watch for deleted-but-open files** → space not freed after `rm` → restart the holding process.
5. **Check inodes** → `df -i` if space is free but writes fail.

---

## Scenario: Root Filesystem 100% Full

### Problem
`/` is full; the system misbehaves.

### Symptoms
`No space left on device`, failed writes, services crashing, can't install packages.

### Possible Causes
Runaway logs, large temp files, old kernels/packages, a big application data dir, deleted-but-open files.

### Commands to Check
```bash
df -h                                 # which FS is full
du -sh /* 2>/dev/null | sort -h       # biggest top-level dirs
du -sh /var/* 2>/dev/null | sort -h   # often /var/log
sudo du -ah /var/log | sort -h | tail # biggest log files
sudo lsof | grep deleted              # deleted-but-open files
df -i                                 # inode exhaustion?
```

### Step-by-Step Fix
1. `df -h` → confirm which filesystem (usually `/`).
2. `du -sh /* | sort -h` then drill down into the biggest (often `/var`).
3. Safely remove obvious junk:
   - Truncate a huge active log: `sudo truncate -s 0 /var/log/huge.log` (keeps the file handle valid).
   - Clean package cache: `sudo apt clean` (or `dnf clean all`).
   - Remove old logs/journals: `sudo journalctl --vacuum-size=200M`.
4. If `lsof | grep deleted` shows space held by a deleted file → **restart that process** to release it.
5. If `df -i` shows inodes full → delete large numbers of small junk files.
6. Re-check `df -h`.

### Prevention
Log rotation, journald size limits, monitoring/alerts at 80–90%, separate `/var` or `/data` volume.

## 7. Commands (toolbox)

```bash
df -h ; df -i
du -sh /path/* | sort -h
sudo truncate -s 0 /var/log/file.log     # empty a growing log safely
sudo journalctl --vacuum-size=200M       # shrink systemd journal
sudo apt clean                           # clear apt cache
sudo lsof | grep deleted                 # find phantom space users
```

## 8. Command Explanation

- `truncate -s 0 file` → empties a file **in place**; safer than `rm` for a log a process is actively writing (a process holding a deleted file keeps consuming space).
- `journalctl --vacuum-size=200M` → caps the systemd journal size.
- `apt clean` / `dnf clean all` → removes cached package downloads.
- `lsof | grep deleted` → reveals space held by files that were deleted but are still open.

## 9. Practice Tasks

1. `df -h` and identify your fullest filesystem.
2. `du -sh /var/* 2>/dev/null | sort -h` and find the biggest.
3. Create a junk file `fallocate -l 100M /tmp/junk`, see `df -h` change, then `rm /tmp/junk`.
4. `journalctl --disk-usage` to see journal size.

## 10. Common Mistakes

- `rm`-ing a log a process is writing → space isn't freed (use `truncate` or restart the process).
- Deleting files under `/boot`, `/etc`, `/usr` blindly to free space — can break the system.
- Treating an inode-exhaustion problem as a space problem.

## 11. Troubleshooting

- **Deleted files but space didn't return** → `lsof | grep deleted`; restart the holding process.
- **Free space but writes fail** → `df -i` (inodes); clear lots of small files.
- **Can't even run commands (`/tmp` full)** → free a little space first (truncate a log) to regain room.

## 12. Best Practices

- Configure **log rotation** (logrotate) and journald size limits.
- Monitor disk usage and alert before 90%.
- Keep application data on a separate volume from the OS.

## 13. Quick Recap

- `df -h` → find FS; `du -sh | sort -h` → find dir; clean safely.
- Use `truncate` for active logs; restart processes holding deleted files.
- Check `df -i` for inodes. Prevent with rotation + monitoring.

## 14. References

- `man df`, `man du`, `man truncate`, `man journalctl`
- [log-cleanup-basics.md](./log-cleanup-basics.md)
