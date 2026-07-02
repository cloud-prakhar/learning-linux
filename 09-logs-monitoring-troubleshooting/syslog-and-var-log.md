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

## 5. How It Works Under the Hood

The reason `/var/log` is *organized by purpose* (not one giant file) comes from the **syslog protocol's two-part tagging**: every message carries a **facility** (who sent it — `auth`, `cron`, `daemon`, `kern`, `mail`, ...) and a **severity** (how bad — emerg=0 … err=3 … info=6 … debug=7).

- **rsyslog routes by those tags.** Its config (`/etc/rsyslog.conf`, `/etc/rsyslog.d/`) has rules like `auth,authpriv.* /var/log/auth.log` — meaning "messages with facility auth go to auth.log." That's *why* login/sudo/SSH events land in `auth.log` while general messages go to `syslog`: the facility tag decided the destination. Understanding this makes "which file?" predictable rather than memorized.
- **Apps that write their own files bypass rsyslog.** nginx/mysql open `/var/log/nginx/error.log` etc. directly — they're not using the syslog facility system, which is why those live in app subdirectories and why their format differs.
- **Rotation renames files behind you.** logrotate (Module 08) turns `syslog` into `syslog.1`, then `syslog.2.gz`, etc. So *today's* events are in `syslog`, but yesterday's may be in `syslog.1` and last week's in a **gzipped** `syslog.2.gz`. That's the crucial gotcha: plain `grep` on a `.gz` file returns binary garbage — you need **`zgrep`**, which decompresses on the fly. Searching "all of last month" often means `zgrep pattern /var/log/syslog*`.
- **Permissions gate the sensitive ones.** `auth.log`/`secure` are readable only by root/`adm` — deliberately, since they reveal login patterns — hence `sudo`.

So the layout is a direct consequence of syslog's facility/severity tagging + rsyslog routing + logrotate compression; each explains where a log is and how to read it.

## 6. Diagram

```mermaid
flowchart LR
    Apps[Apps & system] -->|facility.severity| Rsyslog[rsyslog routes by rule]
    Rsyslog -->|auth.*| Auth[/var/log/auth.log]
    Rsyslog -->|*.*| Sys[/var/log/syslog]
    Web[nginx - writes directly] --> Nginx[/var/log/nginx/*.log]
    Sys -->|logrotate| Rot[syslog.1, syslog.2.gz - use zgrep]
```

## 7. Real-World Examples

**1. The everyday case.** Brute-force SSH attempts? `grep "Failed password" /var/log/auth.log | wc -l` counts them and shows the source IPs — the basis of basic intrusion detection.

**2. Counting attacks, including rotated logs:**

```
$ sudo grep -c "Failed password" /var/log/auth.log
842                                                   # today's failures
$ sudo zgrep -c "Failed password" /var/log/auth.log.*.gz   # older, rotated + gzipped
118
$ sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -3
   611 203.0.113.45                                   # top attacking IP
   150 198.51.100.9
```

Note the **`zgrep`** for the `.gz` archives (Section 5) — a plain `grep` there would return binary noise.

**3. War story — the evidence in the file nobody checked.** After a security review, a team was asked "when did this account first log in from that IP?" The current `auth.log` only went back two days (logrotate). The answer lived in `auth.log.4.gz` from three weeks earlier. `sudo zgrep "Accepted.*<ip>" /var/log/auth.log*` (globbing *all* rotations, gzipped included) surfaced the first login instantly. Lesson: incident history spans rotated logs — always search `logfile*` with `zgrep`, not just the live file.

## 8. Worked Walkthrough

Read, filter, and search across rotations the right way:

```
$ ls -lh /var/log/auth.log*                    # see the rotation set
-rw-r----- 1 syslog adm  2.1M ... /var/log/auth.log
-rw-r----- 1 syslog adm  1.0M ... /var/log/auth.log.1
-rw-r----- 1 syslog adm  180K ... /var/log/auth.log.2.gz     # gzipped
$ sudo tail -n 3 /var/log/auth.log             # recent live entries
Jul  2 09:15:22 web01 sshd[8123]: Accepted publickey for deploy from 198.51.100.7
$ sudo grep -i "sudo" /var/log/auth.log | tail -1   # who used sudo
Jul  2 09:20:01 web01 sudo: alice : TTY=pts/0 ; COMMAND=/usr/bin/systemctl restart nginx
$ sudo zgrep "Failed password" /var/log/auth.log*   # search live + rotated + .gz together
/var/log/auth.log:Jul 2 09:01:12 ... Failed password for root from 203.0.113.45
/var/log/auth.log.2.gz:Jun 20 ... Failed password for admin from 198.51.100.9
```

The key habit: `auth.log*` + `zgrep` searches the *whole* history (live + rotated + compressed), not just today (Section 5).

## 9. Commands

```bash
ls -lh /var/log
sudo tail -n 50 /var/log/syslog                 # recent system messages
sudo tail -f /var/log/auth.log                  # follow logins live
sudo grep "Failed password" /var/log/auth.log   # failed SSH logins
sudo less /var/log/nginx/error.log              # scroll web errors
sudo grep -i error /var/log/syslog | tail
sudo zgrep "error" /var/log/syslog*             # search rotated (gzipped) logs too
```

Sample output for each (dummy values, for reference):

```text
$ ls -lh /var/log
-rw-r----- 1 syslog adm  120M Jul  2 09:14 syslog
-rw-r----- 1 syslog adm   40M Jul  2 09:14 auth.log
drwxr-xr-x 2 root   adm  4.0K Jul  2 09:14 nginx

$ sudo tail -n 2 /var/log/syslog
Jul  2 09:15:01 web01 CRON[9001]: (root) CMD (backup.sh)
Jul  2 09:15:22 web01 sshd[8123]: Accepted publickey for deploy

$ sudo grep -c "Failed password" /var/log/auth.log
842

$ sudo zgrep -c "error" /var/log/syslog.2.gz
53
```

## 10. Command Explanation

- `tail -n 50` / `tail -f` → recent or live entries.
- `grep "Failed password" auth.log` → classic security check (facility `auth` — Section 5).
- `less error.log` → scroll/search a big web log (nginx writes it directly).
- `zgrep` → like `grep` but reads `.gz` rotated logs without unzipping them — essential for history.
- Glob `logfile*` to include rotated files in one search.

## 11. In Production (DevOps Context)

- **Security monitoring** parses `auth.log`/`secure` for failed logins and privilege use — the raw material for fail2ban and SIEM alerts (the war story) (Module 12).
- **Web/app debugging** lives in `/var/log/nginx/`, `/var/log/mysql/`, etc.; these app logs are often the fastest path to a 5xx root cause.
- **Log shipping** (rsyslog can forward over the network; Fluent Bit/Filebeat tail `/var/log`) centralizes text logs alongside journald for fleet-wide search.
- **Rotation awareness** matters: retention policies and incident forensics both depend on knowing logs span `logfile.N[.gz]` (Module 08).

## 12. Practice Tasks

1. `ls -lh /var/log` — note paths for your distro and any `.gz` rotations.
2. `sudo tail -n 20 /var/log/auth.log` (or `/var/log/secure`).
3. `sudo grep -i "sudo" /var/log/auth.log | tail`.
4. Search across rotations: `sudo zgrep "Failed password" /var/log/auth.log*`.

## 13. Common Mistakes

- Assuming Ubuntu paths on RHEL (and vice versa) — `syslog`/`messages`, `auth.log`/`secure`.
- Forgetting `sudo` for `auth.log`/`secure`.
- Using `grep` (not `zgrep`) on rotated `.gz` logs and getting binary noise (Section 5).
- Searching only the live file and missing history in `logfile.N.gz` (the war story).

## 14. Troubleshooting

- **File missing** → wrong distro path or journald-only system (use `journalctl`).
- **Log not updating** → rsyslog may be down (`systemctl status rsyslog`) or the app logs to the journal instead.
- **Binary garbage from grep** → it's a `.gz`; use `zgrep`.
- **Huge log filling disk** → see [Log Cleanup Basics](../08-storage-and-disk-management/log-cleanup-basics.md).

## 15. Best Practices

- Use `grep`/`zgrep` + time context to filter; glob `logfile*` for full history.
- Know auth vs system vs app log locations (driven by facility routing).
- Rotate logs (logrotate) so `/var/log` doesn't fill the disk (Module 08).

## 16. Connects To

- **Prev:** [journalctl Basics](journalctl-basics.md). **Next:** [CPU, Memory & Disk Checks](cpu-memory-disk-checks.md).
- **The two logging worlds:** [Linux Logs Overview](linux-logs-overview.md).
- **Reading/searching tools:** [Viewing Files](../03-files-and-directories/view-files.md), [Searching Files](../03-files-and-directories/search-files.md).
- **Rotation & compression:** [Log Cleanup Basics](../08-storage-and-disk-management/log-cleanup-basics.md).
- **Failed logins → security:** [SSH Basics](../12-linux-security-basics/ssh-basics.md), [Security Best Practices](../12-linux-security-basics/security-best-practices.md).

## 17. Quick Recap

- `/var/log` holds text logs routed by syslog **facility** (auth→auth.log, general→syslog); apps like nginx write their own files.
- Logs rotate into `logfile.N` and gzipped `logfile.N.gz` — use **`zgrep`** and glob `logfile*` to search history.
- `tail`/`grep`/`less`/`zgrep` read them; paths differ by distro family; sensitive logs need `sudo`.

## 18. References

- `man rsyslog.conf`, `man logger`, `man zgrep`
- rsyslog: https://www.rsyslog.com/doc/

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [journalctl Basics](journalctl-basics.md) | ⬆️ Module: [Module 09 — Logs, Monitoring & Troubleshooting](README.md) | ➡️ Next: [CPU, Memory, and Disk Checks](cpu-memory-disk-checks.md) |
