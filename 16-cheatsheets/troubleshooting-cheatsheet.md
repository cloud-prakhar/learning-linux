# Troubleshooting Cheatsheet

## Symptom → First Commands

| Symptom | Run These | Likely Cause |
|---------|-----------|--------------|
| Server slow | `uptime`, `top`, `free -h`, `iostat -xz 1 3` | CPU/mem/IO pressure |
| High CPU | `ps aux --sort=-%cpu \| head` | Runaway process |
| High memory / OOM | `free -h`, `dmesg \| grep -i oom` | Leak / limit |
| Disk full | `df -h`, `du -sh /* \| sort -h`, `lsof \| grep deleted` | Logs/temp/deleted-open |
| No inodes left | `df -i` | Too many small files |
| Service down | `systemctl status svc`, `journalctl -u svc -e` | Config/port/perm |
| Port in use | `ss -ltnp \| grep :PORT`, `lsof -i :PORT` | Conflicting process |
| Can't connect | `getent hosts`, `ping`, `ss -ltnp`, `curl -v` | DNS/port/firewall |
| DNS fails | `dig @8.8.8.8 name`, `cat /etc/resolv.conf` | Resolver issue |
| Permission denied | `id`, `ls -l`, `namei -l /path` | Owner/perm |
| Cron not running | `crontab -l`, `grep CRON /var/log/syslog` | Env/path/logging |
| SSH refused | `systemctl status ssh`, firewall, key perms | Service/firewall/key |

## Error Message → Meaning → Fix

| Error | Meaning | Fix |
|-------|---------|-----|
| `Permission denied` | Lacking rwx | `chmod`/`chown` or `sudo` |
| `command not found` | Typo / not installed | Check spelling / install pkg |
| `No such file or directory` | Wrong path | `pwd`, Tab-complete, absolute path |
| `Connection refused` | No listener | Start service / fix bind |
| `Connection timed out` | Firewall/route | Open port / check route |
| `Address already in use` | Port taken | `ss -ltnp`; stop holder |
| `No space left on device` | Disk or inodes full | `df -h`/`df -i`; clean up |
| `Operation not permitted` | Needs root / immutable | `sudo`; `chattr -i` |
| `UNPROTECTED PRIVATE KEY` | Key too open | `chmod 600 key` |

## Quick Resource Snapshot (one-liners)

```bash
uptime; free -h; df -h                # health at a glance
ps aux --sort=-%cpu | head -5         # top CPU
ps aux --sort=-%mem | head -5         # top memory
du -sh /var/* 2>/dev/null | sort -h   # biggest dirs under /var
ss -ltnp                              # listening ports + owners
systemctl --failed                    # failed services
journalctl -p err --since '30 min ago' --no-pager | tail
```

## The Incident Loop

```
1. Note symptom + time
2. Check resources (top/free/df)
3. Check logs around that time (journalctl --since / /var/log)
4. Form hypothesis -> test -> fix
5. Verify -> Prevent (rotation/monitoring/limits)
```

## Safe Space-Reclaim

| Action | Command |
|--------|---------|
| Empty an active log | `truncate -s 0 file.log` |
| Shrink journal | `journalctl --vacuum-size=200M` |
| Clear pkg cache | `apt clean` / `dnf clean all` |
| Free deleted-open file | restart the holding process |

> Source: Modules [04](../04-users-groups-permissions/), [07](../07-networking-basics/), [08](../08-storage-and-disk-management/), [09](../09-logs-monitoring-troubleshooting/).

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Process & Service Cheatsheet](process-service-cheatsheet.md) | ⬆️ Module: [Module 16 — Cheatsheets](README.md) | ➡️ Next: [Module 17 — Interview & Revision](../17-interview-and-revision/README.md) |
