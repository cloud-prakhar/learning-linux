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

## 5. How It Works Under the Hood

A "full disk" is really one of **three distinct conditions**, and the fix differs for each — which is why blind deleting so often fails:

- **Blocks exhausted (the usual case).** The filesystem's data-block budget is used up (`df -h` = 100%). Real files are too big/too many. Fix: remove or truncate the largest offenders.
- **Inodes exhausted (the sneaky case).** The filesystem ran out of *file slots*, not space (`df -h` shows free, but `df -i` = 100%). Millions of tiny files did it. Fix: delete large *numbers* of small files — size is irrelevant here (see [df/du](df-du-lsblk.md)).
- **Deleted-but-open files (the maddening case).** You `rm`'d a big file but `df` didn't budge. Because `rm` only unlinks the *name* while a process holds the file *open*, the blocks stay allocated until that process closes it (see [Create/Copy/Move/Delete](../03-files-and-directories/create-copy-move-delete.md)). Fix: **restart the holding process** (or truncate instead of delete).

This is also why **`truncate -s 0 file` beats `rm` for an active log.** `rm` on a log a service is writing hits the deleted-but-open trap — space never frees and, worse, the service keeps writing to a now-nameless file. `truncate` empties the file *in place*, so the process keeps its valid file handle and space is reclaimed immediately. And there's a chicken-and-egg trap: at 100% you sometimes can't even run tools or edit files — so free a *little* first (truncate one big log) to regain room to work.

Diagnosis order therefore mirrors the three conditions: `df -h` (blocks?) → `df -i` (inodes?) → `lsof +L1` / `lsof | grep deleted` (orphaned?).

## 6. Diagram

```mermaid
flowchart TD
    F[Disk full symptoms] --> DF{df -h at 100%?}
    DF -->|yes| DU[du -sh path/* | sort -h → find + truncate/remove]
    DF -->|no, but writes fail| DI[df -i → inodes full → delete many small files]
    DU --> Chk{df freed?}
    Chk -->|no| LSOF[lsof +L1: deleted-but-open → restart holder]
    Chk -->|yes| Done[Prevent: rotation + monitoring]
```

## 7. Real-World Examples

**1. The everyday case.** `df -h` shows `/` full → `du -sh /var/* | sort -h` shows `/var/log` is 30G → truncate the giant log and add rotation. Two commands to the culprit.

**2. The three conditions, distinguished by output:**

```
$ df -h /
/dev/nvme0n1p1  40G   40G     0 100% /          # blocks exhausted → normal case
$ df -i /
Filesystem     Inodes IUsed IFree IUse%
/dev/nvme0n1p1 2.6M   2.6M     0  100% /          # OR inodes exhausted → different fix
$ rm /var/log/huge.log ; df -h / | tail -1
/dev/nvme0n1p1  40G   40G     0 100% /          # df UNCHANGED after rm → deleted-but-open
$ sudo lsof +L1 | grep huge
java  711 app 9u REG 0 /var/log/huge.log (deleted)   # the culprit still holds it open
```

Same "disk full," three different signatures → three different fixes (Section 5).

**3. War story — the `rm` that made it worse.** A DB server hit 100%. An engineer `rm`'d a 30 GB MySQL slow-query log to free space. `df` didn't move — MySQL still had it open, kept writing to the invisible inode, and the disk stayed full while the log kept "growing" with no filename. Meanwhile the freed name gave a false sense the problem was handled. The correct move was `truncate -s 0 /var/log/mysql/slow.log` (space back instantly, handle stays valid) or restarting MySQL. They learned the rule: **never `rm` an active log — truncate it.**

## 8. Worked Walkthrough

Trigger, diagnose, and safely reclaim space:

```
$ fallocate -l 500M /tmp/junk.bin          # create a big file to simulate usage
$ df -h /tmp | tail -1
tmpfs           1.9G  500M  1.4G  27% /tmp
$ du -sh /tmp/* 2>/dev/null | sort -h | tail -1
500M    /tmp/junk.bin                       # found the offender
$ rm /tmp/junk.bin && df -h /tmp | tail -1  # nothing holds it open → space returns
tmpfs           1.9G  1.2M  1.9G   1% /tmp
# For an ACTIVE log a service holds open, truncate instead of rm:
$ sudo truncate -s 0 /var/log/app/app.log   # empties in place; handle stays valid
$ sudo journalctl --vacuum-size=200M        # cap the systemd journal too
Vacuuming done, freed 1.2G of archived journals.
$ df -h /                                    # re-check
```

Note the contrast: `rm` worked for `/tmp/junk.bin` (nothing had it open), but for the live app log we used `truncate` — the Section 5 rule in action.

## 9. Commands

```bash
df -h ; df -i                            # blocks and inodes
du -sh /path/* | sort -h                 # localize the big directory
sudo du -ah /var/log | sort -h | tail    # biggest files under a dir
sudo truncate -s 0 /var/log/file.log     # empty a growing log SAFELY (not rm)
sudo journalctl --vacuum-size=200M       # shrink systemd journal
sudo apt clean                           # clear apt cache (dnf clean all on RHEL)
sudo lsof +L1                            # find deleted-but-open (phantom) space
```

Sample output for each (dummy values, for reference):

```text
$ df -h /
/dev/nvme0n1p1  40G   40G     0 100% /

$ df -i /
/dev/nvme0n1p1 2.6M 412K 2.2M 16% /

$ du -sh /var/* 2>/dev/null | sort -h | tail -2
2.4G    /var/cache
30G     /var/log

$ sudo truncate -s 0 /var/log/app/app.log
# (no output; file now 0 bytes, space reclaimed, app keeps writing)

$ sudo journalctl --vacuum-size=200M
Deleted archived journal .../system@....journal (1.0G).
Vacuuming done, freed 1.0G.

$ sudo lsof +L1 | head -2
COMMAND PID USER FD TYPE NLINK NAME
mysqld  810 mysql 12u REG 0 /var/log/mysql/slow.log (deleted)
```

## 10. Command Explanation

- `df -h` / `df -i` → distinguish a block-full from an inode-full disk (Section 5).
- `du -sh /path/* | sort -h` → ranks directories so the biggest surfaces at the bottom.
- `truncate -s 0 file` → empties a file **in place**; the safe choice for a log a process is actively writing (avoids the deleted-but-open trap).
- `journalctl --vacuum-size` → caps the systemd binary journal.
- `apt clean` / `dnf clean all` → removes cached package downloads.
- `lsof +L1` (or `lsof | grep deleted`) → reveals space held by deleted-but-open files → restart that process.

## 11. In Production (DevOps Context)

- **Disk-full is a top-tier incident:** databases go read-only, apps 500, deploys fail, logins can break. Speed comes from the three-condition method, not panic-deleting.
- **`truncate` vs `rm`** is production-critical for live logs (the war story) — many outages get *worse* from `rm`-ing an open log.
- **Kubernetes `DiskPressure`** on a node is this exact problem (usually `/var/lib/containerd` full of images/logs); the node cordons and evicts pods (Module 13).
- **Prevention is the real fix:** logrotate + journald caps + monitoring alerts at 80–90% + separate `/var`/`/data` volumes (next topic).
- **CI runners** fill disks with build caches/artifacts; cleanup steps and ephemeral runners prevent recurring failures.

## 12. Practice Tasks

1. `df -h` and identify your fullest filesystem; `df -i` to check inodes.
2. `du -sh /var/* 2>/dev/null | sort -h` and find the biggest.
3. `fallocate -l 100M /tmp/junk`, see `df -h` change, then `rm /tmp/junk` and confirm space returns.
4. `journalctl --disk-usage`, then `sudo journalctl --vacuum-time=7d`.
5. Practice the safe path: create a log, have `tail -f` hold it open, `rm` it (space not freed), then `truncate`/kill `tail` and observe.

## 13. Common Mistakes

- `rm`-ing a log a process is writing → space isn't freed (the war story); use `truncate` or restart the process.
- Deleting files under `/boot`, `/etc`, `/usr` blindly to free space — can break the system.
- Treating an inode-exhaustion problem as a space problem (check `df -i`).
- Forgetting you may need to free a *little* space first before tools/editors will run at 100%.

## 14. Troubleshooting

**Scenario — Root filesystem 100% full**
- **Symptoms:** `No space left on device`, failed writes, services crashing, can't install packages.
- **Causes:** runaway logs, large temp files, old kernels/packages, big app data dir, deleted-but-open files, or inode exhaustion.
- **Check:**
  ```bash
  df -h ; df -i
  du -sh /* 2>/dev/null | sort -h
  du -sh /var/* 2>/dev/null | sort -h
  sudo lsof +L1
  ```
- **Fix:** ① Confirm the filesystem (`df -h`). ② Drill with `du` to the biggest dir/file. ③ Reclaim safely: `truncate -s 0` a huge active log; `apt clean`/`dnf clean all`; `journalctl --vacuum-size=200M`. ④ If `lsof +L1` shows deleted-but-open → **restart that process**. ⑤ If `df -i` full → delete many small junk files. ⑥ Re-check `df -h`.
- **Prevention:** logrotate, journald size limits, alerts at 80–90%, separate `/var`/`/data` volume.

## 15. Best Practices

- Configure **log rotation** (logrotate) and journald size limits (next topic).
- Monitor disk usage and inodes; alert before 90%.
- Keep application data on a separate volume from the OS.
- Reflex: `truncate` (not `rm`) for active logs; check `df -i` and `lsof +L1` when the obvious fix doesn't work.

## 16. Connects To

- **Prev:** [mount and umount](mount-and-umount.md). **Next:** [Log Cleanup Basics](log-cleanup-basics.md).
- **The three conditions' mechanics:** [df/du/lsblk](df-du-lsblk.md), [Create, Copy, Move, Delete](../03-files-and-directories/create-copy-move-delete.md).
- **Prevention (rotation):** [Log Cleanup Basics](log-cleanup-basics.md), [journalctl Basics](../09-logs-monitoring-troubleshooting/journalctl-basics.md).
- **Node DiskPressure:** [Linux for Kubernetes](../13-real-world-linux-for-devops/linux-for-kubernetes.md).
- **Practice:** [Lab 05 — Disk Full Scenario](../14-hands-on-labs/lab-05-disk-full-scenario.md). **Quick lookup:** [Troubleshooting Cheatsheet](../16-cheatsheets/troubleshooting-cheatsheet.md).

## 17. Quick Recap

- Three conditions, three fixes: **blocks full** (`df -h`) → remove/truncate big files; **inodes full** (`df -i`) → delete many small files; **deleted-but-open** (`lsof +L1`) → restart the holder.
- `truncate -s 0` beats `rm` for active logs; free a little first if 100% blocks your tools.
- Prevent with rotation + journald caps + monitoring + separate data volumes.

## 18. References

- `man df`, `man du`, `man truncate`, `man journalctl`, `man lsof`
- [log-cleanup-basics.md](./log-cleanup-basics.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [mount and umount](mount-and-umount.md) | ⬆️ Module: [Module 08 — Storage & Disk Management](README.md) | ➡️ Next: [Log Cleanup Basics](log-cleanup-basics.md) |
