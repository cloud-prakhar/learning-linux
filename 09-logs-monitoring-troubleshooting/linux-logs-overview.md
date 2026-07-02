# Linux Logs Overview

## 1. What Is This?

A map of Linux logging: the **systemd journal** (binary, queried with `journalctl`) and traditional **text logs** under `/var/log`.

## 2. Why Is This Needed?

Logs are the primary evidence when diagnosing problems. Knowing *where* each kind of log lives saves huge amounts of time during incidents.

## 3. Simple Layman Explanation

Logs are the server's **diary**. Some entries are kept in a structured digital database (journald), others as plain notebooks in a drawer (`/var/log`). To solve a mystery, you read the relevant diary around the time things went wrong.

## 4. Technical Explanation

| Source | Where | Read with |
|--------|-------|-----------|
| systemd journal | binary store (`/var/log/journal` or RAM) | `journalctl` |
| System log | `/var/log/syslog` (Debian) / `/var/log/messages` (RHEL) | `less`, `tail`, `grep` |
| Auth log | `/var/log/auth.log` (Debian) / `/var/log/secure` (RHEL) | `less`, `grep` |
| Kernel | `dmesg` / journal | `dmesg`, `journalctl -k` |
| App logs | `/var/log/<app>/` (e.g., nginx) | `tail`, `less` |

Most modern distros use **journald**, often alongside rsyslog writing text files.

## 5. How It Works Under the Hood

Modern Linux often has *two logging systems running at once*, and knowing how they relate stops the confusion of "why are my logs in two places?":

- **Where logs originate.** Programs emit log messages one of three ways: (a) the kernel writes to its **ring buffer** (a fixed-size in-memory circular log — `dmesg`/`journalctl -k`); (b) services write to **stdout/stderr**, which systemd captures into the **journal**; (c) apps write **directly to their own files** under `/var/log/<app>/` (nginx, mysql) because they manage their own logging.
- **journald is the modern hub.** systemd's `journald` collects the kernel buffer, service stdout/stderr, and traditional syslog messages into **one structured, indexed binary store**. "Structured" means each entry carries *fields* (`_SYSTEMD_UNIT`, `PRIORITY`, `_PID`, timestamp) — so you can query "errors from nginx in the last 10 minutes" precisely, instead of grepping text and hoping.
- **rsyslog often runs alongside it** and, on many distros, *also* writes the classic plain-text files (`/var/log/syslog`, `auth.log`). That's the redundancy: the same login event may appear in *both* the journal (via `journalctl -u ssh`) *and* `/var/log/auth.log`. Neither is "wrong" — they're two views fed by the same stream.
- **Why some logs vanish on reboot.** If `/var/log/journal/` exists, the journal is **persistent** (survives reboots); if not, it lives in RAM (`/run`) and is **lost on reboot**. Text files in `/var/log` persist on disk regardless. This is why "the logs from before the crash are gone" happens — a volatile journal.

So the map is: kernel buffer + service stdout + syslog → journald (structured, queryable) *and* often → `/var/log` text files (grep-able). Pick the tool that matches where the log landed.

## 6. Diagram

```mermaid
flowchart LR
    Kernel[Kernel ring buffer] --> J[(journald - structured, indexed)]
    Services[Service stdout/stderr] --> J
    Rsys[rsyslog] --> J
    Rsys --> Files[/var/log/*.log - text]
    App[nginx/mysql] --> AppFiles[/var/log/app/*]
    J --> Jc[journalctl - query by field]
    Files & AppFiles --> Tools[less / tail / grep]
```

## 7. Real-World Examples

**1. The everyday case.** SSH logins failing? Check `/var/log/auth.log` (or `journalctl -u ssh`). Web 502 errors? `/var/log/nginx/error.log`. Kernel killed a process? `dmesg | grep -i oom`. Each symptom has a home.

**2. The same event in two places:**

```
$ sudo journalctl -u ssh -n 1 --no-pager
Jul 02 09:15:22 web01 sshd[8123]: Accepted publickey for deploy from 198.51.100.7
$ sudo tail -n 1 /var/log/auth.log
Jul  2 09:15:22 web01 sshd[8123]: Accepted publickey for deploy from 198.51.100.7
```

Identical entry via `journalctl` (structured) and `/var/log/auth.log` (text) — the dual-system reality from Section 5.

**3. War story — "the logs from the crash are gone."** After an unexpected reboot, an engineer ran `journalctl -b -1` (previous boot) to find the cause — and got *"Specified boot ID is invalid / no persistent journal."* The journal was **volatile** (no `/var/log/journal/`), so everything before the reboot lived in RAM and evaporated (Section 5). They couldn't diagnose the original crash. The fix going forward: `mkdir -p /var/log/journal` + set `Storage=persistent`, so future crashes leave evidence. On servers you must audit, persistent logging isn't optional.

## 8. Worked Walkthrough

Take inventory of both worlds and confirm persistence:

```
$ ls -lh /var/log | sort -k5 -h | tail -4      # biggest text logs
-rw-r----- 1 syslog  adm  120M ... syslog
-rw-r----- 1 root    adm  240M ... nginx/access.log
drwxr-sr-x 3 root systemd-journal 512M ... journal      # journal IS persistent (dir exists)
$ journalctl --disk-usage
Archived and active journals take up 512.0M in the file system.
$ journalctl -b --no-pager | head -2           # this boot
Jul 02 06:00:01 web01 kernel: Linux version 5.15.0-105-generic ...
$ journalctl -b -1 --no-pager | head -1        # PREVIOUS boot (works only if persistent)
Jul 01 22:14:55 web01 systemd[1]: Shutting down.
$ dmesg | tail -2                              # kernel ring buffer
[ 4211.02] EXT4-fs (nvme0n1p1): mounted filesystem
```

Because `/var/log/journal/` exists, `journalctl -b -1` returned previous-boot logs — the difference between diagnosing a crash and the war story.

## 9. Commands

```bash
ls -lh /var/log                      # what logs exist & their sizes
journalctl -e                        # jump to end of the journal
journalctl -b -1                     # logs from the PREVIOUS boot (if persistent)
tail -n 50 /var/log/syslog           # last 50 system log lines
sudo tail -f /var/log/auth.log       # follow auth events live
dmesg | tail                         # recent kernel messages
sudo grep -i error /var/log/syslog   # find errors
```

Sample output for each (dummy values, for reference):

```text
$ ls -lh /var/log
-rw-r----- 1 syslog adm  120M Jul  2 09:14 syslog
-rw-r----- 1 root   adm   40M Jul  2 09:14 auth.log
drwxr-sr-x 3 root systemd-journal 4.0K Jul 1 journal

$ journalctl -e --no-pager | tail -1
Jul 02 09:15:23 web01 systemd[1]: Started Session 42 of user deploy.

$ dmesg | tail -1
[ 4211.021] EXT4-fs (nvme0n1p1): re-mounted. Opts: (null)

$ sudo grep -ci error /var/log/syslog
17
```

## 10. Command Explanation

- `ls -lh /var/log` → inventory of log files and sizes (also spot huge ones — Module 08).
- `journalctl -e` → opens the journal at the newest entries.
- `journalctl -b -1` → previous boot's logs — the key to diagnosing a crash/reboot (needs persistent journal).
- `tail -f` → live-follow a text log during an incident.
- `dmesg` → kernel ring buffer (hardware, OOM kills, drivers).
- `grep -i error` → case-insensitive search for error lines.

## 11. In Production (DevOps Context)

- **Centralized logging** ships both journald and `/var/log` streams to ELK/Loki/CloudWatch/Splunk, so you query one place across a fleet — the structured fields from Section 5 make this powerful.
- **Persistent journals** are enabled on servers you must audit (the war story) so crashes leave evidence.
- **Containers log to stdout/stderr** by design; the runtime captures it (`docker logs`, `kubectl logs`) — the same "service stdout → collector" model as journald (Module 13).
- **Knowing the distro paths** (`syslog` vs `messages`, `auth.log` vs `secure`) matters when you SSH into an unfamiliar box mid-incident.

## 12. Practice Tasks

1. `ls -lh /var/log` and note the biggest files.
2. `sudo tail -n 20 /var/log/syslog` (or `journalctl -e`).
3. Check persistence: does `/var/log/journal/` exist? Try `journalctl -b -1`.
4. `dmesg | tail` and `sudo grep -i "fail" /var/log/auth.log | tail`.

## 13. Common Mistakes

- Looking in the wrong distro's path (`syslog` vs `messages`, `auth.log` vs `secure`).
- Reading entire logs instead of filtering by time/keyword.
- Expecting pre-reboot logs from a volatile (RAM-only) journal (the war story).
- Forgetting `sudo` for protected logs like `auth.log`.

## 14. Troubleshooting

- **No `/var/log/syslog`** → you're on RHEL (use `/var/log/messages`) or a journald-only system (use `journalctl`).
- **`journalctl -b -1` fails** → the journal isn't persistent; create `/var/log/journal` and set `Storage=persistent`.
- **Logs too big to open** → use `tail`/`grep`/`less`, never `cat` (Module 03).
- **Permission denied** → prepend `sudo` (or join the `systemd-journal` group).

## 15. Best Practices

- Learn your distro's log paths; know which tool reads which store.
- Filter by service/time/keyword from the start.
- Enable persistent journals on servers you must audit; ship logs off-host for fleets.
- Correlate timestamps across multiple logs.

## 16. Connects To

- **Prev:** [Module 09 — Logs, Monitoring & Troubleshooting](README.md). **Next:** [journalctl Basics](journalctl-basics.md).
- **Query the journal:** [journalctl Basics](journalctl-basics.md); **text files:** [Syslog & /var/log](syslog-and-var-log.md).
- **Reading logs (tools):** [Viewing Files](../03-files-and-directories/view-files.md), [Searching Files](../03-files-and-directories/search-files.md).
- **Huge logs / rotation:** [Log Cleanup Basics](../08-storage-and-disk-management/log-cleanup-basics.md).

## 17. Quick Recap

- Two worlds fed by one stream: journald (structured, `journalctl`) and `/var/log` text files (`tail`/`grep`).
- Kernel buffer + service stdout + syslog → journald; apps also write their own `/var/log/<app>` files.
- Journals may be volatile (lost on reboot) unless `/var/log/journal/` exists — enable persistence to survive crashes.

## 18. References

- `man journalctl`, `man dmesg`, `man rsyslogd`
- systemd journal: https://www.freedesktop.org/software/systemd/man/journalctl.html

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 09 — Logs, Monitoring & Troubleshooting](README.md) | ⬆️ Module: [Module 09 — Logs, Monitoring & Troubleshooting](README.md) | ➡️ Next: [journalctl Basics](journalctl-basics.md) |
