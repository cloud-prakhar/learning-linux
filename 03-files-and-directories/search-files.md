# Searching Files

## 1. What Is This?

Two kinds of search: **find files** by name/type/size (`find`, `locate`) and **find text inside files** (`grep`).

## 2. Why Is This Needed?

Servers have thousands of files. You need to locate a specific config, find large files filling a disk, or search logs for an error message — fast.

## 3. Simple Layman Explanation

- `find` = walk the building and check every room for what you want (thorough, live).
- `locate` = look it up in a pre-made index (instant, but may be slightly outdated).
- `grep` = read inside the documents and highlight every line containing your word.

## 4. Technical Explanation

| Tool | Searches | Notes |
|------|----------|-------|
| `find` | Filesystem by name/size/time/type | Real-time, powerful filters |
| `locate` | Prebuilt database of filenames | Fast; run `updatedb` to refresh |
| `grep` | Text **inside** files | Supports regex, recursion |

## 5. How It Works Under the Hood

The three tools differ in *what* they read, which explains their speed and freshness trade-offs:

- **`find` walks the directory tree live.** Starting at the path you give, it reads each directory's entries and `stat()`s files to check name, size, time, and type — then recurses into subdirectories. Because it touches real metadata on disk, results are always current, but scanning a huge tree (like `find /`) is slow and hits directories you may not have permission to read (those "Permission denied" lines).
- **`locate` doesn't touch the tree at all.** It queries a **prebuilt database** of filenames (refreshed by `updatedb`, usually via a daily cron job). That's why it's near-instant — but also why a file created five minutes ago won't appear until the database updates. It knows *names only*, not current sizes or contents.
- **`grep` opens files and scans their bytes** for a pattern, line by line, using a **regular-expression** engine. `-r` makes it walk a directory tree like `find`, opening each file. So `grep -r` cost = number of files × their size; narrowing the path or file glob is what keeps it fast.

Rule of thumb from this: know the **name** and want it now → `locate`. Need **attributes** (size/time/type) or guaranteed-fresh → `find`. Need what's **inside** → `grep`.

## 6. Diagram

```mermaid
flowchart LR
    Q{What are you searching?} --> N[A file by name -> find/locate]
    Q --> T[Text inside files -> grep]
    N --> F[find: live tree walk]
    N --> L[locate: prebuilt index]
```

## 7. Real-World Examples

**1. The everyday case.** "Where's the error?" → `grep -ri "connection refused" /var/log/`. "What's filling the disk?" → `find /var -size +100M`. These two patterns solve a huge share of daily problems.

**2. Hunting an error with line numbers:**

```
$ grep -rn "connection refused" /var/log/ 2>/dev/null
/var/log/app/error.log:8123:2026-07-02 09:14:02 ERROR connection refused to db:5432
/var/log/app/error.log:8140:2026-07-02 09:14:07 ERROR connection refused to db:5432
```

`-n` gives file:line so you can jump straight there; `2>/dev/null` hides the permission-denied noise (Section 5).

**3. War story — the disk detective.** A server alerted at 95% disk. `df -h` said `/var` was the culprit but not *what*. One command found it:

```
$ find /var -type f -size +500M -exec ls -lh {} \; 2>/dev/null
-rw-r--r-- 1 root root 11G Jul  2 03:00 /var/lib/docker/containers/abc.../abc-json.log
```

An unrotated Docker container log had grown to 11 GB. `find` with a size filter located it in seconds where clicking through folders would have taken ages — the "what's filling the disk?" pattern from Section 4 (Module 08).

## 8. Worked Walkthrough

Find files by attribute, then search inside them:

```
$ find /etc -name "*.conf" -type f 2>/dev/null | head -3
/etc/ssh/ssh_config
/etc/logrotate.conf
/etc/nginx/nginx.conf
$ find /etc -name "*.conf" -mtime -7 2>/dev/null       # changed in the last 7 days
/etc/nginx/nginx.conf
$ grep -n "listen" /etc/nginx/nginx.conf
34:    listen 80 default_server;
35:    listen [::]:80 default_server;
$ grep -rin "worker_connections" /etc/nginx/ 2>/dev/null
/etc/nginx/nginx.conf:12:    worker_connections 768;
```

You located `.conf` files (by name and type), narrowed to recently changed ones (by time), then searched *inside* one for a setting — the exact chain you use when auditing a server's config.

## 9. Commands

```bash
find /etc -name "*.conf"           # files ending in .conf under /etc
find . -type d -name "logs"        # directories named 'logs' here
find /var/log -size +50M           # files larger than 50 MB
find /home -mtime -1               # files modified in last 1 day
grep "error" app.log               # lines containing 'error'
grep -i "error" app.log            # case-insensitive
grep -r "TODO" .                   # search recursively in all files here
grep -rn "timeout" /etc            # show file + line numbers
locate nginx.conf                  # fast name lookup (needs mlocate/plocate)
```

Sample output for each (dummy values, for reference):

```text
$ find /etc -name "*.conf" | head -3
/etc/logrotate.conf
/etc/resolv.conf
/etc/nginx/nginx.conf

$ find /var/log -size +50M
/var/log/journal/system.journal

$ find /home -mtime -1
/home/alice/notes.txt

$ grep -in "error" app.log
42:2026-07-02 09:14:02 ERROR db timeout
88:2026-07-02 09:20:11 error: retry limit reached

$ grep -rn "timeout" /etc/nginx/ 2>/dev/null
/etc/nginx/nginx.conf:29:    keepalive_timeout 65;

$ locate nginx.conf
/etc/nginx/nginx.conf
```

## 10. Command Explanation

- `find PATH -name "PATTERN"` → search by name; quotes prevent the shell expanding `*` before `find` sees it.
- `-type d` / `-type f` → directories / files only.
- `-size +50M` → larger than 50 MB (`+` greater, `-` smaller).
- `-mtime -1` → modified within the last day.
- `grep -i` → ignore case; `-r` recursive; `-n` show line numbers; combine as `-rin`.
- `locate` → instant filename search from a database; refresh with `sudo updatedb`.

## 11. In Production (DevOps Context)

- **Disk-full triage** leans on `find ... -size +N` to locate runaway files/logs (the war story) (Module 08).
- **Log analysis** is `grep`/`grep -r` across `/var/log` — the fastest path from "something's wrong" to the offending line (Module 09).
- **`grep` in pipelines/CI** filters command output (`... | grep -q PATTERN`) to make pass/fail decisions.
- **`find -exec`** performs bulk operations (delete old files, fix permissions) across a tree — the backbone of cleanup scripts (Modules 08, 10).
- Faster modern variants (`ripgrep`/`rg`, `fd`) exist on dev machines; the concepts here map directly.

## 12. Practice Tasks

1. `find /etc -name "*.conf" 2>/dev/null | head`.
2. `grep -rin "root" /etc/passwd`.
3. `find . -type f -size +1M` in a large directory.
4. `find ~ -mtime -1` to see what you changed today.
5. Combine: `find /var/log -name "*.log" -size +10M 2>/dev/null` to spot big logs.

## 13. Common Mistakes

- Forgetting quotes around `*.conf`, so the shell expands it first.
- Running `find /` without filters — slow and noisy. Narrow the path.
- Expecting `locate` to find brand-new files (its DB may be stale — Section 5).
- Forgetting `-r` with `grep` on a directory (it needs recursion to descend).

## 14. Troubleshooting

- **`grep: No such file`** → wrong path; add `-r` to search a directory tree.
- **`locate: command not found`** → install `mlocate`/`plocate`, then `sudo updatedb`.
- **`find` permission-denied lines** → harmless; add `2>/dev/null` to hide them.
- **`locate` misses a new file** → run `sudo updatedb`, or use `find` for guaranteed-fresh results.

## 15. Best Practices

- Combine: `find . -name "*.log" -exec grep -l "ERROR" {} \;`.
- Narrow `find` to a specific directory for speed.
- Use `grep -rn` to jump straight to the file and line.

## 16. Connects To

- **Prev:** [Viewing Files](view-files.md). **Next:** [Links — Hard & Soft](links-hard-soft.md).
- **View what you find:** [Viewing Files](view-files.md) (`less`, `tail`).
- **Disk-full hunting:** [df/du/lsblk](../08-storage-and-disk-management/df-du-lsblk.md), [Disk Full Troubleshooting](../08-storage-and-disk-management/disk-full-troubleshooting.md).
- **Searching logs:** [Linux Logs Overview](../09-logs-monitoring-troubleshooting/linux-logs-overview.md).

## 17. Quick Recap

- `find` = live tree walk by attributes (name/size/time/type); always fresh, slower on big trees.
- `locate` = instant name lookup from a prebuilt index; may be stale.
- `grep` = scan text inside files with regex; `-rin` is the everyday combo.
- Name+now → `locate`; attributes/fresh → `find`; contents → `grep`.

## 18. References

- `man find`, `man grep`, `man locate`
- GNU grep: https://www.gnu.org/software/grep/manual/

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Viewing Files](view-files.md) | ⬆️ Module: [Module 03 — Files & Directories](README.md) | ➡️ Next: [Links — Hard and Soft (Symbolic)](links-hard-soft.md) |
