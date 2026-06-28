# Production Server Checklist

## 1. What Is This?

A consolidated **go-live checklist** for a Linux server, drawing on every module: setup, security, services, storage, logging, and automation.

## 2. Why Is This Needed?

Production servers must be secure, reliable, and observable from day one. A checklist prevents the "we forgot to set up backups/monitoring" disasters that hit real teams.

## 3. Simple Layman Explanation

Before a restaurant opens to the public, the owner checks the kitchen, locks, fire alarms, and supplies. This is that pre-opening checklist for a server.

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

## 5. Real-World Example

A team launches a new API server. Before traffic: keys-only SSH, ufw allowing 22/443, unattended security upgrades, Nginx as `www-data` enabled at boot, `/data` on its own volume with disk alerts, logrotate configured, nightly off-host backups, and a one-page runbook. It survives its first incident calmly because everything was in place.

## 6. Diagram

```mermaid
flowchart LR
    Access[SSH keys + firewall] --> Patch[Updated]
    Patch --> Svc[Services enabled]
    Svc --> Store[Storage + monitoring]
    Store --> Logs[Logging + rotation]
    Logs --> Backup[Backups tested]
    Backup --> Go[Go live]
```

## 7. Commands

```bash
# Access & firewall (Module 12)
sudo grep -E "PermitRootLogin|PasswordAuthentication" /etc/ssh/sshd_config
sudo ufw status verbose

# Updates (Module 6/12)
sudo apt update && apt list --upgradable

# Services (Module 5)
systemctl list-units --type=service --state=running
systemctl is-enabled nginx

# Storage & health (Module 8/9)
df -h ; df -i ; free -h ; uptime
du -sh /var/log

# Backups & automation (Module 11)
crontab -l
timedatectl                       # time sync status
```

## 8. Command Explanation

- The SSH `grep` → confirms root/password login are disabled.
- `ufw status verbose` → verifies default-deny + allowed ports.
- `systemctl ... --state=running` / `is-enabled` → ensures only intended services run and survive reboot.
- `df`/`free`/`uptime` → baseline health snapshot.
- `crontab -l` → confirms backups/cleanups are scheduled.
- `timedatectl` → confirms clock sync (critical for logs and TLS).

## 9. Practice Tasks

1. Run the checklist commands against a test server and tick each item.
2. Identify one gap and fix it (e.g., enable a firewall or schedule a backup).
3. Write a one-page runbook for the server (restart steps, log locations, owner).

## 10. Common Mistakes

- Going live without backups or monitoring.
- Leaving default SSH/firewall settings.
- No runbook — incidents become guesswork.
- Clock not synced, breaking log correlation and certificates.

## 11. Troubleshooting

- Use the relevant module's troubleshooting section per area (security → 12, disk → 8, services → 5, logs → 9).
- If unsure where to start during an incident, follow the incident loop in [Module 09 scenarios](../09-logs-monitoring-troubleshooting/real-world-troubleshooting-scenarios.md).

## 12. Best Practices

- Automate the checklist where possible (config management).
- Re-audit periodically, not just at launch.
- Keep the runbook current and accessible.
- Test backups and failure scenarios before you need them.

## 13. Quick Recap

- Cover access, updates, services, storage, logging, monitoring, backups, time, and docs.
- Every item maps back to a module you've completed.
- A checklist + runbook turns chaos into routine.

## 14. References

- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks
- [Module 15 Project 05: troubleshooting playbook](../15-mini-projects/project-05-troubleshooting-playbook.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Linux for CI/CD](linux-for-ci-cd.md) | ⬆️ Module: [Module 13 — Real-World Linux for DevOps](README.md) | ➡️ Next: [Module 14 — Hands-On Labs](../14-hands-on-labs/README.md) |
