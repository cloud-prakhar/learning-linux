# Process & Service Cheatsheet

## Viewing Processes

| Command | Purpose | Example |
|---------|---------|---------|
| `ps aux` | All processes, detailed | `ps aux` |
| `ps aux --sort=-%cpu` | Top CPU users | `ps aux --sort=-%cpu \| head` |
| `ps aux --sort=-%mem` | Top memory users | `ps aux --sort=-%mem \| head` |
| `ps -ef \| grep x` | Find by name | `ps -ef \| grep nginx` |
| `pgrep -a name` | PIDs by name | `pgrep -a sshd` |
| `pstree` | Process tree | `pstree` |
| `top` / `htop` | Live monitor | `top` (P=cpu, M=mem, q=quit) |

## Job Control

| Command | Purpose |
|---------|---------|
| `cmd &` | Run in background |
| `jobs` | List background jobs |
| `fg %1` | Foreground job 1 |
| `bg %1` | Resume job in background |
| `Ctrl+Z` | Suspend foreground job |
| `$!` | PID of last background job |

## Signals & Kill

| Command | Purpose |
|---------|---------|
| `kill PID` | Graceful (SIGTERM 15) |
| `kill -9 PID` | Force (SIGKILL) — last resort |
| `kill -HUP PID` | Reload config |
| `killall name` | Signal all by name |
| `pkill -f 'pattern'` | Kill by command-line match |
| `kill -l` | List signals |

## systemd Services

| Command | Purpose |
|---------|---------|
| `systemctl status svc` | State + recent logs |
| `sudo systemctl start/stop svc` | Start / stop now |
| `sudo systemctl restart svc` | Restart |
| `sudo systemctl reload svc` | Reload config (no downtime) |
| `sudo systemctl enable --now svc` | Start now + at boot |
| `sudo systemctl disable svc` | Don't start at boot |
| `systemctl is-active svc` | active/inactive |
| `systemctl is-enabled svc` | enabled/disabled |
| `systemctl --failed` | Failed services |
| `sudo systemctl daemon-reload` | After editing unit files |

## Logs (with services)

| Command | Purpose |
|---------|---------|
| `journalctl -u svc -e` | Service logs, newest |
| `journalctl -u svc -f` | Follow live |
| `journalctl -u svc --since '10 min ago'` | Time window |
| `journalctl -p err` | Errors only |
| `journalctl -b -1` | Previous boot |

## Resource Health

| Command | Purpose |
|---------|---------|
| `uptime` / `nproc` | Load vs cores |
| `free -h` | Memory & swap |
| `df -h` / `df -i` | Disk space / inodes |
| `dmesg \| grep -i oom` | Out-of-memory kills |

## Service Debug Flow

```
systemctl status -> journalctl -u svc -e -> validate config (nginx -t) -> fix -> restart
```

> Source: [Module 05](../05-processes-and-services/), [Module 09](../09-logs-monitoring-troubleshooting/).

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Networking Cheatsheet](networking-cheatsheet.md) | ⬆️ Module: [Module 16 — Cheatsheets](README.md) | ➡️ Next: [Troubleshooting Cheatsheet](troubleshooting-cheatsheet.md) |
