# df, du, and lsblk

## 1. What Is This?

The three measurement tools:
- `df` — free space per **filesystem**.
- `du` — disk usage of **directories/files**.
- `lsblk` — list of **block devices** (disks/partitions) and mount points.

## 2. Why Is This Needed?

When space is tight, you need to know *which* filesystem is full (`df`), then *which* directory is eating it (`du`), and *what* devices exist (`lsblk`). This trio solves nearly every disk problem.

## 3. Simple Layman Explanation

- `df` = how full each shelf (filesystem) is.
- `du` = how much each box (directory) on the shelf weighs.
- `lsblk` = the map of all your shelves.

## 4. Technical Explanation

| Tool | Answers | Scope |
|------|---------|-------|
| `df` | How full is each mounted filesystem? | Per-mount |
| `du` | How big is this directory/file? | Per-path |
| `lsblk` | What disks/partitions exist & where mounted? | Devices |

Also: filesystems can run out of **inodes** (file slots) even with free space — check `df -i`.

## 5. Real-World Example

`df -h` shows `/` at 100%. `du -sh /var/* | sort -h` shows `/var/log` is 30G. You found the offender in two commands and can now clean it (next topics).

## 6. Diagram

```mermaid
flowchart LR
    DF[df -h: which FS is full] --> DU[du: which dir is big]
    DU --> Act[Clean / move data]
    LSBLK[lsblk: device map] --> DF
```

## 7. Commands

```bash
df -h                        # free space, human-readable
df -h /var                   # space for the FS holding /var
df -i                        # inode usage (file count limits)
du -sh /var/log              # total size of /var/log
du -sh /var/*                # size of each item under /var
du -sh /var/* | sort -h      # sorted smallest->largest
du -ah /var/log | sort -h | tail   # biggest files under a dir
lsblk                        # device/partition/mount tree
ncdu /var                    # interactive usage browser (install ncdu)
```

## 8. Command Explanation

- `df -h` → human-readable free/used per filesystem; the first thing to run for "disk full".
- `df -i` → inode usage; high here with free space means too many small files.
- `du -sh PATH` → **s**ummary, **h**uman-readable total for a path.
- `du -sh /var/* | sort -h` → ranks subdirectories so the biggest is at the bottom.
- `ncdu` → a friendly interactive explorer (install with `apt install ncdu`).

Expected `df -h`:

```
Filesystem  Size  Used Avail Use% Mounted on
/dev/sda1    49G   47G  0.5G  99% /
```

## 9. Practice Tasks

1. `df -h` — note the `Use%` of `/`.
2. `du -sh /var/* | sort -h` — find the largest directory under `/var`.
3. `df -i` — check inode usage.
4. Install `ncdu` and explore `ncdu /var`.

## 10. Common Mistakes

- Running `du /` without `-s` and drowning in output. Use `-sh` per directory.
- Forgetting `df -i` when `df -h` shows free space but writes still fail.
- Misreading `du` (directory size) as `df` (filesystem free space).

## 11. Troubleshooting

- **"No space left on device" but `df` shows free space** → inodes exhausted; check `df -i` and delete many small files.
- **`du` is slow/permission-denied** → add `sudo` and `2>/dev/null` to skip errors.
- **`df` total ≠ `du` total** → deleted-but-open files still consuming space (see disk-full troubleshooting).

## 12. Best Practices

- Standard drill: `df -h` → `du -sh /path/* | sort -h` → drill into the biggest.
- Monitor disk usage with alerts before it reaches 90%.
- Check both space (`df -h`) and inodes (`df -i`).

## 13. Quick Recap

- `df -h` = filesystem free space; `df -i` = inodes.
- `du -sh dir/*` = directory sizes; sort to find offenders.
- `lsblk` = device map.

## 14. References

- `man df`, `man du`, `man lsblk`
- ncdu: https://dev.yorhel.nl/ncdu
