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

## 5. Real-World Example

"Where's the error?" → `grep -ri "connection refused" /var/log/`. "What's filling the disk?" → `find /var -size +100M`. These two patterns solve a huge share of daily problems.

## 6. Diagram

```mermaid
flowchart LR
    Q{What are you searching?} --> N[A file by name -> find/locate]
    Q --> T[Text inside files -> grep]
```

## 7. Commands

```bash
find /etc -name "*.conf"           # files ending in .conf under /etc
find . -type d -name "logs"        # directories named 'logs' here
find /var/log -size +50M           # files larger than 50 MB
find /home -mtime -1               # files modified in last 1 day
grep "error" app.log               # lines containing 'error'
grep -i "error" app.log            # case-insensitive
grep -r "TODO" .                   # search recursively in all files here
grep -rn "timeout" /etc            # show file + line numbers
locate nginx.conf                  # fast name lookup (needs mlocate)
```

## 8. Command Explanation

- `find PATH -name "PATTERN"` → search by name; quotes prevent the shell expanding `*`.
- `-type d` / `-type f` → directories / files only.
- `-size +50M` → larger than 50 MB (`+` greater, `-` smaller).
- `-mtime -1` → modified within the last day.
- `grep -i` → ignore case; `-r` recursive; `-n` show line numbers; combine as `-rin`.
- `locate` → instant filename search from a database; refresh with `sudo updatedb`.

## 9. Practice Tasks

1. `find /etc -name "*.conf" | head`.
2. `grep -rin "root" /etc/passwd`.
3. `find . -type f -size +1M` in a large directory.
4. `find ~ -mtime -1` to see what you changed today.

## 10. Common Mistakes

- Forgetting quotes around `*.conf`, so the shell expands it first.
- Running `find /` without filters — slow and noisy. Narrow the path.
- Expecting `locate` to find brand-new files (its DB may be stale).

## 11. Troubleshooting

- **`grep: No such file`** → wrong path; add `-r` to search a directory tree.
- **`locate: command not found`** → install `mlocate`/`plocate`, then `sudo updatedb`.
- **`find` permission denied lines** → harmless; add `2>/dev/null` to hide them.

## 12. Best Practices

- Combine: `find . -name "*.log" -exec grep -l "ERROR" {} \;`.
- Narrow `find` to a specific directory for speed.
- Use `grep -rn` to jump straight to the file and line.

## 13. Quick Recap

- `find` = locate files by attributes (name/size/time/type).
- `grep` = find text inside files (`-rin` is the everyday combo).
- `locate` = instant filename lookup from an index.

## 14. References

- `man find`, `man grep`, `man locate`
- GNU grep: https://www.gnu.org/software/grep/manual/
