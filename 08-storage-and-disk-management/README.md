# Module 08 — Storage & Disk Management

## What You Will Learn

- Disk, partition, filesystem, and mount concepts.
- Checking space and usage with df, du, lsblk.
- Mounting and unmounting filesystems.
- Diagnosing and fixing a full disk.
- Safely cleaning up logs and freeing space.

## Why This Module Matters

"Disk full" is one of the most common production incidents — it crashes apps, blocks logins, and breaks deploys. Knowing how to find and free space fast is essential.

## Real-World Use Case

A server stops accepting writes because `/` is 100% full. You find the culprit directory with `du`, clean old logs safely, and prevent recurrence — exactly what this module teaches.

## Topics Covered

| File | What It Covers |
|------|----------------|
| [disk-partition-mount-concepts.md](./disk-partition-mount-concepts.md) | Disks, partitions, filesystems, mounts |
| [df-du-lsblk.md](./df-du-lsblk.md) | Measuring space & usage |
| [mount-and-umount.md](./mount-and-umount.md) | Attaching/detaching storage |
| [disk-full-troubleshooting.md](./disk-full-troubleshooting.md) | Fixing a full disk |
| [log-cleanup-basics.md](./log-cleanup-basics.md) | Safe log cleanup |

## Learning Flow

```mermaid
flowchart LR
    A[Disk/partition/mount concepts] --> B[df/du/lsblk]
    B --> C[mount/umount]
    C --> D[Disk full fix]
    D --> E[Log cleanup]
```

## Hands-On Practice

Inspect your disks with `lsblk`, measure usage with `df -h` and `du -sh`, then find the biggest directories on the system.

## Common Mistakes

- Deleting files that are still open by a process (space isn't freed).
- Confusing disk space (`df`) with inode exhaustion (`df -i`).

## Troubleshooting

- Disk full → `df -h` to find the filesystem, `du` to find the directory.
- Space "missing" after delete → a process still holds the file open.

## Best Practices

- Set up log rotation; monitor disk usage before it hits 100%.
- Never delete files you don't understand under `/`, `/var`, `/boot`.

## Quick Revision

- `lsblk` = devices, `df -h` = free space, `du -sh` = directory sizes.
- Disk-full fix: locate (df) → drill down (du) → clean safely.

## Next Module

➡️ [09 — Logs, Monitoring & Troubleshooting](../09-logs-monitoring-troubleshooting/).
