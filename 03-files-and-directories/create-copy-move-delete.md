# Create, Copy, Move, Delete

## 1. What Is This?

The core file operations: **create** (`touch`, `mkdir`), **copy** (`cp`), **move/rename** (`mv`), and **delete** (`rm`, `rmdir`).

## 2. Why Is This Needed?

You constantly organize files: making config backups, moving logs, creating project folders, removing junk. These five commands cover almost all of it.

## 3. Simple Layman Explanation

This is like managing papers on a desk: make a new sheet (`touch`), make a folder (`mkdir`), photocopy a sheet (`cp`), move/rename it (`mv`), and shred it (`rm`). Remember: the shredder has **no undo**.

## 4. Technical Explanation

| Command | Action | Note |
|---------|--------|------|
| `touch` | Create empty file / update timestamp | |
| `mkdir` | Make directory | `-p` makes parent dirs |
| `cp` | Copy file/dir | `-r` for directories |
| `mv` | Move or rename | Same command for both |
| `rm` | Delete file/dir | `-r` recursive, **permanent** |
| `rmdir` | Delete empty directory | Only if empty |

## 5. How It Works Under the Hood

The behaviors that surprise beginners all come from one fact: **a filename is just an entry in a directory pointing to an inode** (the real data). Recall from [Linux Filesystem Overview](../02-linux-basics/linux-file-system-overview.md) that directories map names → inodes.

- **`cp` genuinely duplicates data.** It creates a new inode and copies the bytes, so a copy uses real disk space and takes time proportional to size. Copying a 10 GB file writes 10 GB.
- **`mv` within the same filesystem is instant** — it doesn't move bytes at all; it just rewrites directory entries (unlink the old name, link the new one). Rename a 10 GB file and it's done in milliseconds. But `mv` *across* filesystems (e.g., `/` to a USB) can't just relink — it must **copy then delete**, which is why moving to another disk is slow and can half-finish.
- **`rm` doesn't erase data; it removes a directory entry and decrements the inode's link count.** When the count hits zero *and* no process holds the file open, the space is freed. This is why there's no recycle bin (the name is simply gone) — and, importantly, why deleting a huge log file that a running process still has open does **not** free space until that process closes it (a classic Module 08 disk-full gotcha).

So `mv` = relink (fast), `cp` = copy bytes (real space), `rm` = unlink (permanent, but space only frees when the last link and last open handle are gone.)

## 6. Diagram

```mermaid
flowchart LR
    T[touch/mkdir: create] --> C[cp: copy bytes]
    C --> M[mv: relink name]
    M --> R[rm: unlink - permanent]
```

## 7. Real-World Examples

**1. The everyday case — back up before you edit.** Before editing Nginx config: `cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`. If your edit breaks the server, you `mv` the backup back. This habit has saved countless engineers.

**2. Watching `mv` rename vs. cross-disk copy:**

```
$ time mv bigfile.iso /home/alice/bigfile.iso      # same filesystem: just relinks
real    0m0.003s
$ time mv bigfile.iso /mnt/usb/bigfile.iso         # different filesystem: copy + delete
real    0m18.442s
```

Same command, wildly different time — exactly the relink-vs-copy distinction from Section 5.

**3. War story — `rm` that didn't free the disk.** A server hit 100% disk. An engineer ran `rm /var/log/app/huge.log` (40 GB) — but `df -h` still showed the disk full. The app had the file *open* and kept writing to the now-nameless inode, so the space never freed (Section 5). `df` stayed at 100% until they restarted the app (or truncated the file with `: > huge.log`). Understanding that `rm` only unlinks turned a "why won't the disk clear?!" panic into a known pattern (Module 08).

## 8. Worked Walkthrough

Build, copy, rename, and delete — watching link counts and space:

```
$ mkdir -p lab/data && cd lab
$ touch data/file{1,2,3}.txt
$ ls data
file1.txt  file2.txt  file3.txt
$ cp -r data data_backup            # real duplication (new inodes)
$ ls
data  data_backup
$ mv data/file1.txt data/renamed.txt   # instant: same-fs relink
$ ls data
file2.txt  file3.txt  renamed.txt
$ rm -i data/file2.txt              # -i asks first (safe while learning)
rm: remove regular empty file 'data/file2.txt'? y
$ rm -r data_backup                 # recursive delete of a directory tree
$ ls
data
```

Notice `mv` renamed without copying, and `rm -r` removed a whole tree in one step. There is no undo after that `y` — which is why `-i` and a prior `ls`/`pwd` matter.

## 9. Commands

```bash
touch notes.txt                 # create empty file / update timestamp
mkdir project                   # create directory
mkdir -p project/src/utils      # create nested dirs in one go
cp notes.txt notes.bak          # copy a file
cp -r project project_copy      # copy a directory (recursive)
mv notes.bak archive/           # move file into a directory
mv old.txt new.txt              # rename a file
rm notes.txt                    # delete a file
rm -r project_copy              # delete a directory and contents
rm -i important.txt             # ask before deleting
rmdir empty_dir                 # delete an empty directory
```

Sample output for each (dummy values, for reference):

```text
$ mkdir -p project/src/utils
# (no output = success)

$ cp -r project project_copy
# (no output = success)

$ ls project_copy/src
utils

$ mv old.txt new.txt
# (no output = success; old.txt is gone, new.txt exists)

$ rm -i important.txt
rm: remove regular file 'important.txt'? y

$ rmdir empty_dir
# (errors if the dir is not empty: "rmdir: failed to remove: Directory not empty")
```

## 10. Command Explanation

- `mkdir -p a/b/c` → `-p` creates all missing parent directories without error.
- `cp -r` → `-r` (recursive) is required to copy directories; it copies bytes into new inodes.
- `mv` → if the target is a directory it moves; if it's a new name it renames (a relink on the same filesystem).
- `rm -r` → deletes recursively. **No recycle bin** — it unlinks names permanently.
- `rm -i` → prompts `y/n` before each delete; safe while learning.

## 11. In Production (DevOps Context)

- **Config backups** (`cp file file.bak`) before edits are standard change-management hygiene on servers.
- **`rm` + open file handle** is a frequent disk-full incident cause: the fix is often to restart the writing process, not to delete more (Module 08).
- **`mv` for atomic deploys:** writing a new file then `mv`-ing it over the old name is atomic on the same filesystem, so readers never see a half-written file — a common safe-deploy trick.
- **`rm -rf` in scripts/CI** with an unset variable (`rm -rf $DIR/` where `$DIR` is empty → `rm -rf /`) is a legendary disaster; always quote and guard (Module 10).

## 12. Practice Tasks

1. `mkdir -p lab/data` then `touch lab/data/file{1,2,3}.txt`.
2. `cp -r lab lab_backup`.
3. `mv lab/data/file1.txt lab/data/renamed.txt`.
4. `rm -i lab/data/file2.txt` and confirm.
5. `rm -r lab_backup`.
6. (Concept) `ls -li` a file, make a copy, and confirm the copy has a *different* inode number.

## 13. Common Mistakes

- `rm -rf` on the wrong path. Always double-check with `pwd` and `ls` first.
- Forgetting `-r` when copying/deleting directories.
- Overwriting a file with `cp`/`mv` silently — use `-i`.
- Expecting `rm` to instantly free disk space while a process still holds the file open (the war story).

## 14. Troubleshooting

- **"cannot remove: Is a directory"** → use `rm -r` (or `rmdir` if empty).
- **"Permission denied"** → you lack write permission on the *directory* (Module 04); may need `sudo`.
- **Deleted a big file but disk still full** → a process holds it open; restart it or use `: > file` to truncate (Module 08).
- **Accidentally overwrote a file** → restore from a backup; there's no built-in undo.

## 15. Best Practices

- Back up before editing: `cp file file.bak`.
- Use `-i` while learning; never blind-run `rm -rf`.
- Use `mkdir -p` to avoid "directory exists" errors in scripts.
- In scripts, quote variables and guard paths so an empty variable can't turn `rm -rf` catastrophic.

## 16. Connects To

- **Prev:** [Module 03 — Files & Directories](README.md). **Next:** [Viewing Files](view-files.md).
- **Why rm is permanent (inodes):** [Linux Filesystem Overview](../02-linux-basics/linux-file-system-overview.md), [Links — Hard & Soft](links-hard-soft.md).
- **The disk-full "rm didn't help" case:** [Disk Full Troubleshooting](../08-storage-and-disk-management/disk-full-troubleshooting.md).
- **Safe deletes in scripts:** [Log Cleanup Script Example](../10-shell-scripting/log-cleanup-script-example.md).

## 17. Quick Recap

- Create: `touch`, `mkdir -p`. Copy: `cp -r` (real bytes). Move/rename: `mv` (relink, fast on same fs). Delete: `rm -r` (unlink, permanent).
- `rm` only frees space when the last link *and* last open handle are gone.
- Always verify the path with `pwd`/`ls` before deleting.

## 18. References

- GNU Coreutils: https://www.gnu.org/software/coreutils/manual/
- `man cp`, `man mv`, `man rm`, `man mkdir`

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 03 — Files & Directories](README.md) | ⬆️ Module: [Module 03 — Files & Directories](README.md) | ➡️ Next: [Viewing Files](view-files.md) |
