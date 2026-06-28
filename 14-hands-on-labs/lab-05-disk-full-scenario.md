# Lab 05 — Disk-Full Scenario

## Objective

Simulate a directory filling with large files, locate the culprit with `df`/`du`, and reclaim space safely — including the "deleted-but-open file" trap.

## Scenario

An app directory is ballooning and threatens the disk. You'll practice the exact drill used in production: locate → drill down → clean safely.

## Prerequisites

- A Linux environment with some free space (this lab creates ~300 MB of junk in `/tmp`).
- Modules 08 and 09 read.

## Steps & Commands

```bash
# 1. Baseline
df -h /
mkdir -p /tmp/disklab/app/logs

# 2. Create large junk files (simulate growth)
fallocate -l 150M /tmp/disklab/app/big1.dat
fallocate -l 100M /tmp/disklab/app/logs/app.log
fallocate -l 50M  /tmp/disklab/app/logs/old.log

# 3. Locate the heavy directory
du -sh /tmp/disklab/* | sort -h
du -sh /tmp/disklab/app/* | sort -h
du -ah /tmp/disklab | sort -h | tail -5     # biggest files

# 4. The "deleted-but-open" trap: a process holds a log open
tail -f /tmp/disklab/app/logs/app.log >/dev/null &
TAIL_PID=$!
rm /tmp/disklab/app/logs/app.log            # delete while it's open
du -sh /tmp/disklab/app/logs                 # size may not drop fully
sudo lsof +L1 2>/dev/null | grep deleted | head   # space held by deleted file

# 5. Release the space the right way
kill "$TAIL_PID"                             # stop the holder -> space freed
sleep 1
du -sh /tmp/disklab/app/logs

# 6. Safely empty a (still-open) growing log instead of rm
fallocate -l 80M /tmp/disklab/app/logs/live.log
truncate -s 0 /tmp/disklab/app/logs/live.log
du -sh /tmp/disklab/app/logs/live.log
```

## Expected Output (samples)

```
$ du -sh /tmp/disklab/app/* | sort -h
100M  /tmp/disklab/app/logs
150M  /tmp/disklab/app/big1.dat

$ sudo lsof +L1 | grep deleted
tail  4321 you  3r  REG  ...  104857600  /tmp/disklab/app/logs/app.log (deleted)

$ du -sh /tmp/disklab/app/logs/live.log
0  /tmp/disklab/app/logs/live.log
```

## Explanation

- `fallocate -l 150M file` → instantly creates a large file to simulate usage.
- `du -sh dir/* | sort -h` → ranks directories/files by size (the core locate step, Module 08).
- The **trap**: `rm` on a file a process still has open does **not** free space until the process closes it — proven by `lsof ... grep deleted`.
- Killing the holder releases the space.
- `truncate -s 0` → empties an active log **in place**, the safe way to reclaim space without breaking the writer.

## Validation

- `du` identifies `big1.dat` / `logs` as the largest.
- After killing `TAIL_PID`, the logs directory size drops.
- `live.log` is `0` bytes after `truncate`.

## Cleanup

```bash
kill "$TAIL_PID" 2>/dev/null
rm -rf /tmp/disklab
df -h /
```

## Troubleshooting

- **`fallocate` not supported** → use `dd if=/dev/zero of=file bs=1M count=150`.
- **Space didn't free after rm** → that's the lesson; find the holder with `lsof +L1 | grep deleted` and kill/restart it.
- **`lsof` permission denied** → run with `sudo`.
- **Low free space to begin with** → reduce the file sizes in step 2.
