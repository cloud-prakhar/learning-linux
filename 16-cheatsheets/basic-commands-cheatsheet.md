# Basic Commands Cheatsheet

## Navigation

| Command | Purpose | Example | Real-World Use |
|---------|---------|---------|----------------|
| `pwd` | Print current directory | `pwd` | "Where am I?" before acting |
| `ls -lah` | List all, long, human sizes | `ls -lah /var` | Inspect a directory fully |
| `ls -lt` | Sort by time, newest first | `ls -lt /var/log` | Find recently changed files |
| `cd` | Change directory | `cd /etc` | Move around |
| `cd -` | Previous directory | `cd -` | Toggle between two dirs |
| `tree -L 2` | Show structure | `tree -L 2 ~` | Visualize a project |

## Files & Directories

| Command | Purpose | Example |
|---------|---------|---------|
| `touch f` | Create empty file / update time | `touch app.log` |
| `mkdir -p a/b` | Make nested dirs | `mkdir -p proj/src` |
| `cp -r src dst` | Copy (recursive) | `cp -r site site.bak` |
| `mv a b` | Move / rename | `mv old.txt new.txt` |
| `rm -r dir` | Delete recursively (permanent!) | `rm -r tmpdir` |
| `rm -i f` | Delete with confirmation | `rm -i important` |
| `ln -s tgt link` | Symbolic link | `ln -s /opt/app/v2 current` |

## Viewing Files

| Command | Purpose | Example |
|---------|---------|---------|
| `cat f` | Print whole file | `cat /etc/os-release` |
| `less f` | Scroll a big file (`q` quits) | `less /var/log/syslog` |
| `head -n N f` | First N lines | `head -n 5 file` |
| `tail -n N f` | Last N lines | `tail -n 50 file` |
| `tail -f f` | Follow live (logs) | `tail -f app.log` |

## Searching

| Command | Purpose | Example |
|---------|---------|---------|
| `find P -name 'x'` | Find files by name | `find /etc -name '*.conf'` |
| `find P -size +50M` | Find large files | `find /var -size +50M` |
| `find P -mtime -1` | Modified in last day | `find ~ -mtime -1` |
| `grep -rin 'txt' P` | Search text (recursive, line#) | `grep -rin error /var/log` |
| `locate name` | Fast filename lookup | `locate nginx.conf` |

## Archiving

| Command | Purpose | Example |
|---------|---------|---------|
| `tar -czf a.tgz dir/` | Create gzip archive | `tar -czf logs.tgz /var/log/app` |
| `tar -xzf a.tgz` | Extract | `tar -xzf logs.tgz` |
| `tar -tzf a.tgz` | List contents | `tar -tzf logs.tgz` |
| `zip -r a.zip dir/` | Create zip | `zip -r proj.zip proj/` |

## Help & Misc

| Command | Purpose |
|---------|---------|
| `man cmd` | Manual page (`q` to quit) |
| `cmd --help` | Quick usage |
| `history` | Past commands |
| `clear` / `Ctrl+L` | Clear screen |
| `Ctrl+C` | Cancel running command |
| `which cmd` | Path of a command |

> Source modules: [02](../02-linux-basics/), [03](../03-files-and-directories/).

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 16 — Cheatsheets](README.md) | ⬆️ Module: [Module 16 — Cheatsheets](README.md) | ➡️ Next: [Permissions Cheatsheet](permissions-cheatsheet.md) |
