# Production Server Checklist

## 1. What Is This?

A consolidated **go-live checklist** for a Linux server, drawing on every module: setup, security, services, storage, logging, and automation.

## 2. Why Is This Needed?

Production servers must be secure, reliable, and observable from day one. A checklist prevents the "we forgot to set up backups/monitoring" disasters that hit real teams.

## 3. Simple Layman Explanation

Before a restaurant opens to the public, the owner checks the kitchen, locks, fire alarms, and supplies. This is that pre-opening checklist for a server. And like a restaurant, the point isn't just to pass inspection once — it's that when something goes wrong on a busy night, everything you need is already in place.

## 4. Technical Explanation — The Checklist

| Area | Item | Module |
|------|------|--------|
| Access | SSH keys only; root & password login disabled | 12 |
| Firewall | Default-deny; only needed ports open | 12 |
| Updates | Patched; automatic security updates on | 6, 12 |
| Users | Least privilege; scoped sudo; no shared logins | 4, 12 |
| Services | Needed services `enabled --now`; others disabled | 5 |
| Storage | Adequate disk; separate data volume; usage monitored | 8 |
| Logging | Log rotation; journald limits; centralized if possible | 8, 9 |
| Monitoring | CPU/mem/disk alerts; health checks | 9, 15 |
| Backups | Scheduled, off-host, restore-tested | 10, 11 |
| Time | NTP/chrony in sync (logs/certs depend on it) | — |
| Docs | Runbook: how to restart, where logs are, who owns it | — |

## 5. How It Works Under the Hood

This checklist is the **capstone** — it isn't new material, it's every prior module assembled into one operational whole, ordered by *when each item saves you*:

- **The items form a dependency chain, not a random list.** Access and firewall (Module 12) come first because a server exposed with weak SSH is compromised before you finish the rest. Updates (Module 06) close known holes. Least privilege and non-root services (Modules 04, 12) shrink the blast radius *when* something slips through. Storage, logging, and monitoring (Modules 08, 09) keep it running and *observable*. Backups (Modules 10, 11) are the last line — recovery when everything else failed. Read top to bottom, it's the defense-in-depth story from [security-best-practices](../12-linux-security-basics/security-best-practices.md), extended with reliability and observability.
- **`enabled` ≠ `active` — the reboot trap.** From [systemd-services](../05-processes-and-services/systemd-services.md): a service that's `active` now but not `enabled` is running *until the next reboot*, then silently gone. The checklist item "needed services `enabled --now`" exists because "it's running, ship it" hides a landmine — the box reboots (patch, crash, cloud maintenance) and the service doesn't come back. Verifying `is-enabled` *and* `is-active` is what makes "it works" survive a restart.
- **Two disk numbers, not one.** From [disk-full-troubleshooting](../08-storage-and-disk-management/disk-full-troubleshooting.md): a server can be "out of disk" from **blocks** (`df -h`) *or* **inodes** (`df -i`) — millions of tiny files exhaust inodes while `df -h` still shows free space. The checklist checks *both*, plus a **separate data volume** so a runaway log or upload fills `/data`, not `/`, keeping the OS alive. This is why "monitor disk" means two metrics and a partition strategy, not one `df`.
- **Time sync is a silent prerequisite for everything observable.** If the clock drifts (no NTP/chrony), log timestamps across servers don't line up — correlating an incident becomes guesswork — and TLS certificates can be judged expired/not-yet-valid, breaking HTTPS mysteriously. It's on the checklist precisely *because* nothing screams when it's wrong; you discover it mid-incident when the logs "don't make sense." A synced clock is the assumption every other diagnostic quietly depends on.
- **Backups and runbook encode "recovery is a first-class feature."** An *untested* backup (Module 11) is a guess; the checklist says **restore-tested** because the only proof a backup works is restoring it. The **runbook** — how to restart, where the logs are, who owns it — turns an incident from panicked archaeology into following steps. Both exist because production isn't "does it work on launch day," it's "can a tired on-call engineer recover it at 3 AM" — which is only true if recovery was built in advance.

So the checklist is the whole course applied at once: secure the perimeter, patch, minimize privilege, keep it running and observable, and make recovery routine — each item a specific module's hard-won lesson, arranged so nothing critical is discovered *missing* during an incident.

## 6. Diagram

```mermaid
flowchart LR
    Access[Access + firewall - keep them out] --> Patch[Updated + auto-patch]
    Patch --> Priv[Least privilege - limit damage]
    Priv --> Svc[Services enabled --now - survive reboot]
    Svc --> Store[Storage: df -h AND df -i, separate /data]
    Store --> Obs[Logging + rotation + monitoring + NTP]
    Obs --> Backup[Backups off-host, restore-TESTED]
    Backup --> Doc[Runbook] --> Go[Go live]
```

## 7. Real-World Examples

**1. The everyday case.** A team launches a new API server. Before traffic: keys-only SSH, ufw allowing 22/443, unattended security upgrades, Nginx as `www-data` enabled at boot, `/data` on its own volume with disk alerts, logrotate configured, nightly off-host backups, and a one-page runbook. It survives its first incident calmly because everything was in place.

**2. Running the go-live audit and catching a gap:**

```
$ grep -E '^(PermitRootLogin|PasswordAuthentication)' /etc/ssh/sshd_config   # Access
PermitRootLogin no
PasswordAuthentication no
$ sudo ufw status | head -1                    # Firewall
Status: active
$ systemctl is-enabled nginx ; systemctl is-active nginx   # Services: BOTH must pass
enabled
active
$ df -h / ; df -i /                             # Storage: blocks AND inodes
/dev/xvda1   8.0G  3.1G  4.9G  39% /
/dev/xvda1   524288 68000 456288  13% /         # inodes healthy too
$ timedatectl | grep 'System clock'             # Time
System clock synchronized: yes
$ crontab -l | grep -q backup && echo "backups scheduled" || echo "MISSING: no backups!"
MISSING: no backups!                            # ← gap caught before go-live
```

Every green line is a checklist item *verified*, and the last line caught the classic gap — no backups — *before* traffic, exactly what the checklist is for (Section 5).

**3. War story — the service that vanished on reboot.** A team launched an app, confirmed `systemctl status myapp` was `active`, and went live. Weeks later a routine kernel patch rebooted the box — and the app never came back, causing a 40-minute outage while on-call figured out why. The cause was pure Section 5: the service was `active` but never `enabled`, so it didn't start at boot. A one-line checklist item — verify `systemctl is-enabled`, not just `is-active` — would have caught it. Lesson: "it's running" and "it survives a reboot" are *different facts*; production requires both, and the checklist is what forces you to check the second.

## 8. Worked Walkthrough

Walk the checklist top-to-bottom on a candidate server, fixing gaps as you go:

```
$ # 1. Access & firewall (Module 12)
$ grep -E '^(PermitRootLogin|PasswordAuthentication)' /etc/ssh/sshd_config && sudo ufw status verbose | head -3
PermitRootLogin no
PasswordAuthentication no
Status: active
Default: deny (incoming), allow (outgoing)
$ # 2. Updates (Module 06/12)
$ systemctl is-enabled unattended-upgrades || sudo apt install -y unattended-upgrades
enabled
$ # 3. Services survive reboot (Module 05) — fix the gap if found
$ systemctl is-enabled nginx || sudo systemctl enable nginx
enabled
$ # 4. Storage: both numbers + separate data volume (Module 08)
$ df -h /data && df -i / | tail -1
/dev/xvdf   98G   12G   81G  14% /data
/dev/xvda1  524288 70112 454176  14% /
$ # 5. Time, backups, runbook
$ timedatectl | grep synchronized ; crontab -l | grep backup ; ls RUNBOOK.md
System clock synchronized: yes
0 2 * * * /opt/scripts/backup.sh /data /backups >> /var/log/backup.log 2>&1
RUNBOOK.md
```

Each block maps to a module; the `||` fallbacks *fix* gaps in place (enable the service, install unattended-upgrades) — the checklist as an active audit, not a passive list (Section 5).

## 9. Commands

```bash
# Access & firewall (Module 12)
sudo grep -E "PermitRootLogin|PasswordAuthentication" /etc/ssh/sshd_config
sudo ufw status verbose

# Updates (Module 6/12)
sudo apt update && apt list --upgradable
systemctl is-enabled unattended-upgrades

# Services (Module 5) — check enabled AND active
systemctl list-units --type=service --state=running
systemctl is-enabled nginx ; systemctl is-active nginx

# Storage & health (Module 8/9) — blocks AND inodes
df -h ; df -i ; free -h ; uptime
du -sh /var/log

# Backups, time, docs (Module 11)
crontab -l
timedatectl                       # time sync status
```

Sample output (dummy values, for reference):

```text
$ systemctl is-enabled nginx ; systemctl is-active nginx
enabled
active

$ df -i /
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/xvda1      524288  68000  456288   13% /

$ timedatectl
               Local time: Wed 2026-07-02 11:45:00 UTC
      System clock synchronized: yes
                    NTP service: active

$ crontab -l
0 2 * * * /opt/scripts/backup.sh /data /backups >> /var/log/backup.log 2>&1
```

## 10. Command Explanation

- The SSH `grep` → confirms root/password login are disabled (Module 12).
- `ufw status verbose` → verifies default-deny + allowed ports.
- `systemctl is-enabled` **and** `is-active` → ensures the service runs *and* survives a reboot (the reboot trap — Section 5).
- `df -h` **and** `df -i` → catches both block-full and inode-full (Section 5 / Module 08).
- `free`/`uptime` → baseline health; `du -sh /var/log` → logs aren't silently eating the disk.
- `crontab -l` → confirms backups/cleanups are scheduled; `timedatectl` → clock sync (critical for logs and TLS — Section 5).

## 11. In Production (DevOps Context)

- **The checklist is code, not a wiki page:** Ansible/Terraform apply and *enforce* every item, and a compliance scanner (`lynis`, CIS tools, cloud posture) audits continuously — drift becomes a ticket. A server that can't pass isn't allowed to take traffic.
- **Golden images ship pre-checked:** the base AMI/container already passes access, updates, and service items, so each new instance is production-ready at boot — the checklist is a *build-time* gate (ties to [linux-for-aws](linux-for-aws.md)).
- **Observability is centralized:** logs, metrics, and alerts leave the box (CloudWatch/Prometheus/Loki), so disk/CPU/mem alerts and health checks fire *before* users notice — the monitoring items at fleet scale (Module 09).
- **The runbook lives with the code:** restart steps, log locations, dashboards, and the on-call owner are in the repo, so recovery (the last checklist line) is following documented steps at 3 AM, not archaeology (Module 15's troubleshooting playbook).

## 12. Practice Tasks

1. Run the checklist commands against a test server and tick each item — verify, don't assume.
2. Identify one gap and fix it (e.g., `systemctl enable` a service that was only active, or schedule a backup).
3. Confirm both `df -h` and `df -i` are healthy, and that data lives on a separate volume from `/`.
4. Write a one-page runbook for the server (restart steps, log locations, owner) and confirm `timedatectl` shows the clock synced.

## 13. Common Mistakes

- Going live without backups or monitoring (the gap the checklist exists to catch).
- Confirming a service is `active` but not `enabled` — it vanishes on reboot (the war story).
- Checking only `df -h` and missing inode exhaustion (`df -i`) — Section 5 / Module 08.
- No runbook — incidents become guesswork; clock not synced, breaking log correlation and TLS.

## 14. Troubleshooting

**Service didn't come back after a reboot**
- **Cause:** it was `active` but not `enabled` (the war story).
- **Fix:** `sudo systemctl enable --now <svc>`; audit all critical services with `is-enabled` (Module 05).

**"Disk full" but `df -h` shows free space**
- **Cause:** inode exhaustion or a deleted-but-open file.
- **Fix:** `df -i` for inodes; `lsof | grep deleted` for held files (Module 08).

**Logs across servers don't line up during an incident**
- **Cause:** clock drift — NTP/chrony not running.
- **Fix:** `timedatectl set-ntp true`; verify `System clock synchronized: yes` (Section 5).

**Not sure where to start during an incident**
- **Fix:** each area maps to its module's troubleshooting section (security → 12, disk → 8, services → 5, logs → 9); follow the incident loop in [Module 09 scenarios](../09-logs-monitoring-troubleshooting/real-world-troubleshooting-scenarios.md).

## 15. Best Practices

- Automate the checklist with config management; make passing it a gate for taking traffic.
- Re-audit periodically, not just at launch — verify each control rather than assuming it.
- Check `is-enabled` *and* `is-active`; monitor both `df -h` and `df -i`; keep the clock synced.
- Keep the runbook current and accessible; test backups and failure scenarios *before* you need them.

## 16. Connects To

- **Prev:** [Linux for CI/CD](linux-for-ci-cd.md). **Next:** [Module 14 — Hands-On Labs](../14-hands-on-labs/README.md).
- **Security items:** [Security Best Practices](../12-linux-security-basics/security-best-practices.md), [SSH Basics](../12-linux-security-basics/ssh-basics.md), [Firewall Basics](../12-linux-security-basics/firewall-basics-ufw-firewalld.md), [Least Privilege](../12-linux-security-basics/least-privilege.md).
- **Reliability items:** [systemd Services](../05-processes-and-services/systemd-services.md) (`enabled` vs `active`), [Disk Full Troubleshooting](../08-storage-and-disk-management/disk-full-troubleshooting.md) (`df -h` vs `df -i`), [CPU/Memory/Disk Checks](../09-logs-monitoring-troubleshooting/cpu-memory-disk-checks.md).
- **Recovery items:** [Scheduled Backup Example](../11-automation-and-cron/scheduled-backup-example.md), [Real-World Troubleshooting Scenarios](../09-logs-monitoring-troubleshooting/real-world-troubleshooting-scenarios.md), [Module 15 Troubleshooting Playbook](../15-mini-projects/project-05-troubleshooting-playbook.md).

## 17. Quick Recap

- The checklist is the whole course applied at once, ordered by when each item saves you: access → updates → least privilege → services → storage → observability → backups → docs.
- Verify the *second facts*: `enabled` **and** `active`, `df -h` **and** `df -i`, backups **restore-tested**, clock **synced** — each is a module's hard-won lesson.
- A verified checklist plus a runbook turns a 3 AM incident from chaos into routine.

## 18. References

- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks
- [Module 15 Project 05: troubleshooting playbook](../15-mini-projects/project-05-troubleshooting-playbook.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Linux for CI/CD](linux-for-ci-cd.md) | ⬆️ Module: [Module 13 — Real-World Linux for DevOps](README.md) | ➡️ Next: [Module 14 — Hands-On Labs](../14-hands-on-labs/README.md) |
