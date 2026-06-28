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

## 5. Real-World Example

SSH logins failing? Check `/var/log/auth.log` (or `journalctl -u ssh`). Web 502 errors? `/var/log/nginx/error.log`. Kernel killed a process? `dmesg | grep -i oom`. Each symptom has a home.

## 6. Diagram

```mermaid
flowchart LR
    Kernel[Kernel] --> J[journald]
    Services[Services] --> J
    J --> Jc[journalctl]
    Services --> Files[/var/log/*.log]
    Files --> Tools[less / tail / grep]
```

## 7. Commands

```bash
ls -lh /var/log                      # what logs exist & their sizes
journalctl -e                        # jump to end of the journal
tail -n 50 /var/log/syslog           # last 50 system log lines
sudo tail -f /var/log/auth.log       # follow auth events live
dmesg | tail                         # recent kernel messages
sudo grep -i error /var/log/syslog   # find errors
```

## 8. Command Explanation

- `ls -lh /var/log` → inventory of log files and sizes (also spot huge ones).
- `journalctl -e` → opens the journal at the newest entries.
- `tail -f` → live-follow a text log during an incident.
- `dmesg` → kernel ring buffer (hardware, OOM kills, drivers).
- `grep -i error` → case-insensitive search for error lines.

## 9. Practice Tasks

1. `ls -lh /var/log` and note the biggest files.
2. `sudo tail -n 20 /var/log/syslog` (or `journalctl -e`).
3. `dmesg | tail`.
4. `sudo grep -i "fail" /var/log/auth.log | tail`.

## 10. Common Mistakes

- Looking in the wrong distro's path (`syslog` vs `messages`, `auth.log` vs `secure`).
- Reading entire logs instead of filtering by time/keyword.
- Forgetting `sudo` for protected logs like `auth.log`.

## 11. Troubleshooting

- **No `/var/log/syslog`** → you're on RHEL (use `/var/log/messages`) or a journald-only system (use `journalctl`).
- **Logs too big to open** → use `tail`/`grep`/`less`, never `cat`.
- **Permission denied** → prepend `sudo`.

## 12. Best Practices

- Learn your distro's log paths.
- Filter by service/time/keyword from the start.
- Correlate timestamps across multiple logs.

## 13. Quick Recap

- Two worlds: journald (`journalctl`) and `/var/log` text files.
- auth → logins, nginx → web, dmesg → kernel.
- Filter, don't dump.

## 14. References

- `man journalctl`, `man dmesg`, `man rsyslogd`
- systemd journal: https://www.freedesktop.org/software/systemd/man/journalctl.html

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 09 — Logs, Monitoring & Troubleshooting](README.md) | ⬆️ Module: [Module 09 — Logs, Monitoring & Troubleshooting](README.md) | ➡️ Next: [journalctl Basics](journalctl-basics.md) |
