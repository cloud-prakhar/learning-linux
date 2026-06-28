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

## 5. Real-World Example

Before editing Nginx config: `cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`. If your edit breaks the server, you `mv` the backup back. This habit has saved countless engineers.

## 6. Diagram

```mermaid
flowchart LR
    T[touch/mkdir: create] --> C[cp: copy]
    C --> M[mv: move/rename]
    M --> R[rm: delete]
```

## 7. Commands

```bash
touch notes.txt                 # create empty file
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

## 8. Command Explanation

- `mkdir -p a/b/c` → `-p` creates all missing parent directories without error.
- `cp -r` → `-r` (recursive) is required to copy directories.
- `mv` → if the target is a directory it moves; if it's a new name it renames.
- `rm -r` → deletes recursively. **No recycle bin** — gone is gone.
- `rm -i` → prompts `y/n` before each delete; safe while learning.

## 9. Practice Tasks

1. `mkdir -p lab/data` then `touch lab/data/file{1,2,3}.txt`.
2. `cp -r lab lab_backup`.
3. `mv lab/data/file1.txt lab/data/renamed.txt`.
4. `rm -i lab/data/file2.txt` and confirm.
5. `rm -r lab_backup`.

## 10. Common Mistakes

- `rm -rf` on the wrong path. Always double-check with `pwd` and `ls` first.
- Forgetting `-r` when copying/deleting directories.
- Overwriting a file with `cp`/`mv` silently — use `-i`.

## 11. Troubleshooting

- **"cannot remove: Is a directory"** → use `rm -r` (or `rmdir` if empty).
- **"Permission denied"** → you lack write permission (Module 04); may need `sudo`.
- **Accidentally overwrote a file** → restore from a backup; there's no built-in undo.

## 12. Best Practices

- Back up before editing: `cp file file.bak`.
- Use `-i` while learning; never blind-run `rm -rf`.
- Use `mkdir -p` to avoid "directory exists" errors in scripts.

## 13. Quick Recap

- Create: `touch`, `mkdir -p`. Copy: `cp -r`. Move/rename: `mv`. Delete: `rm -r` (permanent).
- Always verify the path before deleting.

## 14. References

- GNU Coreutils: https://www.gnu.org/software/coreutils/manual/
- `man cp`, `man mv`, `man rm`, `man mkdir`
