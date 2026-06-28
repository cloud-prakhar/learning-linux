# Command Practice Questions

> Recall the command before checking the answer. Cover the answer column with your hand.

## Navigation & Files

| Task | Command |
|------|---------|
| Show current directory | `pwd` |
| List all files with sizes | `ls -lah` |
| Find recently modified files first | `ls -lt` |
| Go to home directory | `cd` or `cd ~` |
| Go back to previous directory | `cd -` |
| Create nested directories | `mkdir -p a/b/c` |
| Copy a directory | `cp -r src dst` |
| Rename a file | `mv old new` |
| Delete a directory and contents | `rm -r dir` |
| Create a symlink | `ln -s target link` |

## Viewing & Searching

| Task | Command |
|------|---------|
| Follow a log live | `tail -f file` |
| First 20 lines | `head -n 20 file` |
| Scroll a big file | `less file` |
| Find `.conf` files under /etc | `find /etc -name '*.conf'` |
| Find files > 100MB | `find / -size +100M` |
| Search text recursively with line numbers | `grep -rn 'text' .` |
| Search ignoring case | `grep -i 'error' file` |

## Permissions

| Task | Command |
|------|---------|
| Make executable | `chmod +x script.sh` |
| Set 644 | `chmod 644 file` |
| Lock down an SSH key | `chmod 600 key` |
| Change owner and group | `sudo chown user:group file` |
| Recursively change ownership | `sudo chown -R u:g dir` |
| Show your groups | `id` |

## Processes & Services

| Task | Command |
|------|---------|
| Top CPU processes | `ps aux --sort=-%cpu \| head` |
| Live process monitor | `top` or `htop` |
| Find a process by name | `pgrep -a nginx` |
| Gracefully kill a PID | `kill PID` |
| Force kill | `kill -9 PID` |
| Service status | `systemctl status svc` |
| Start at boot + now | `sudo systemctl enable --now svc` |
| Service logs (newest) | `journalctl -u svc -e` |
| Failed services | `systemctl --failed` |

## Packages

| Task | Command (apt / dnf) |
|------|---------------------|
| Refresh index | `sudo apt update` / (auto) |
| Install | `sudo apt install pkg` / `sudo dnf install pkg` |
| Upgrade all | `sudo apt upgrade` / `sudo dnf update` |
| Remove | `sudo apt remove pkg` / `sudo dnf remove pkg` |
| Search | `apt search x` / `dnf search x` |

## Networking

| Task | Command |
|------|---------|
| Show IP addresses | `ip a` |
| Resolve a name | `getent hosts example.com` / `dig +short example.com` |
| Test reachability | `ping -c4 8.8.8.8` |
| Get HTTP status code | `curl -s -o /dev/null -w '%{http_code}\n' URL` |
| Listening ports + process | `ss -ltnp` |
| Who owns port 80 | `sudo lsof -i :80` |
| Test a remote port | `nc -zv host 443` |

## Storage

| Task | Command |
|------|---------|
| Free space per filesystem | `df -h` |
| Inode usage | `df -i` |
| Size of directories, sorted | `du -sh */ \| sort -h` |
| List disks/partitions | `lsblk` |
| Empty an active log | `truncate -s 0 file.log` |
| Find deleted-open files | `lsof \| grep deleted` |

## Logs & Health

| Task | Command |
|------|---------|
| Errors in last 30 min | `journalctl -p err --since '30 min ago'` |
| Kernel messages | `dmesg \| tail` |
| OOM kills | `dmesg \| grep -i oom` |
| Load vs cores | `uptime` ; `nproc` |
| Memory & swap | `free -h` |

## Scripting

| Task | Command |
|------|---------|
| Safe script header | `set -euo pipefail` |
| First argument | `$1` |
| All arguments | `"$@"` |
| Default for unset var | `${1:-default}` |
| Debug-trace a script | `bash -x script.sh` |
| Syntax check only | `bash -n script.sh` |

> Source: Modules 02–11. Need the "why"? Open the cheatsheet or topic file for that command.
