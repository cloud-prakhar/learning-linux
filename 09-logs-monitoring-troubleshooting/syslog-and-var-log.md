# syslog and /var/log

## 1. What Is This?

The traditional Linux logging system: a logging daemon (**rsyslog**) writes plain-text logs into **`/var/log`**, organized by purpose.

## 2. Why Is This Needed?

Many systems and apps still write text logs to `/var/log`. They're easy to read with standard tools (`tail`, `grep`, `less`) and are often where app-specific logs (nginx, mysql) live.

## 3. Simple Layman Explanation

`/var/log` is a **drawer of labeled notebooks**: one for system messages, one for logins, one for the web server. You open the relevant notebook and read around the time of the problem.

## 4. Technical Explanation

| File (Debian/Ubuntu) | RHEL equivalent | Contents |
|----------------------|-----------------|----------|
| `/var/log/syslog` | `/var/log/messages` | General system messages |
| `/var/log/auth.log` | `/var/log/secure` | Authentication, sudo, SSH |
| `/var/log/kern.log` | (in messages) | Kernel messages |
| `/var/log/dpkg.log` | `/var/log/yum.log` | Package operations |
| `/var/log/nginx/` | `/var/log/nginx/` | Web server access/error |
| `/var/log/boot.log` | `/var/log/boot.log` | Boot messages |

**syslog severity** ranges from emergency to debug; messages also carry a **facility** (auth, cron, daemon...).

## 5. Real-World Example

Brute-force SSH attempts? `grep "Failed password" /var/log/auth.log | wc -l` counts them and shows the source IPs — the basis of basic intrusion detection.

## 6. Diagram

```mermaid
flowchart LR
    Apps[Apps & system] --> Rsyslog[rsyslog]
    Rsyslog --> Sys[/var/log/syslog]
    Rsyslog --> Auth[/var/log/auth.log]
    Web[nginx] --> Nginx[/var/log/nginx/*.log]
```

## 7. Commands

```bash
ls -lh /var/log
sudo tail -n 50 /var/log/syslog                 # recent system messages
sudo tail -f /var/log/auth.log                  # follow logins live
sudo grep "Failed password" /var/log/auth.log   # failed SSH logins
sudo less /var/log/nginx/error.log              # scroll web errors
sudo grep -i error /var/log/syslog | tail
sudo zgrep "error" /var/log/syslog.1.gz         # search rotated (gzipped) logs
```

## 8. Command Explanation

- `tail -n 50` / `tail -f` → recent or live entries.
- `grep "Failed password" auth.log` → classic security check.
- `less error.log` → scroll/search a big web log.
- `zgrep` → like `grep` but reads `.gz` rotated logs without unzipping them.

## 9. Practice Tasks

1. `ls -lh /var/log` — note paths for your distro.
2. `sudo tail -n 20 /var/log/auth.log` (or `/var/log/secure`).
3. `sudo grep -i "sudo" /var/log/auth.log | tail`.
4. If a `.gz` rotated log exists, search it with `zgrep`.

## 10. Common Mistakes

- Assuming Ubuntu paths on RHEL (and vice versa).
- Forgetting `sudo` for `auth.log`/`secure`.
- Using `grep` (not `zgrep`) on rotated `.gz` logs and getting binary noise.

## 11. Troubleshooting

- **File missing** → wrong distro path or journald-only system (use `journalctl`).
- **Log not updating** → rsyslog may be down (`systemctl status rsyslog`) or the app logs to the journal instead.
- **Huge log** → see [Module 08 log cleanup](../08-storage-and-disk-management/log-cleanup-basics.md).

## 12. Best Practices

- Use `grep`/`zgrep` + time context to filter.
- Know auth vs system vs app log locations.
- Rotate logs (logrotate) so `/var/log` doesn't fill the disk.

## 13. Quick Recap

- `/var/log` holds text logs: syslog/messages (system), auth.log/secure (logins), app dirs.
- `tail`/`grep`/`less`/`zgrep` read them.
- Paths differ by distro family.

## 14. References

- `man rsyslog.conf`, `man logger`
- rsyslog: https://www.rsyslog.com/doc/
