# Final Revision Notes

> A one-page summary of the whole repo. Skim before an interview or to refresh.

## Mental Models

- **Layers:** Hardware → Kernel → Libraries → Shell → Applications.
- **Filesystem:** one tree from `/`; `/etc` configs, `/var/log` logs, `/home` users, `/bin` commands.
- **Paths:** absolute (from `/`) vs relative (from here); scripts prefer absolute.
- **Permissions:** owner/group/others × rwx; r=4 w=2 x=1; `600` keys, `644` files, `755` dirs.
- **Processes:** each has a PID; signals control them; `systemd` manages services.
- **Networking:** IP=address, port=door, DNS=phonebook; debug in layers.
- **Containers/Cloud:** Docker/K8s/EC2 are Linux underneath; same tools apply.

## Daily Command Toolkit

```
Navigate:   pwd, ls -lah, cd, find, grep -rn
Files:      cp -r, mv, rm -r, tar -czf/-xzf, ln -s
Perms:      chmod, chown, id, ls -l, namei -l
Processes:  ps aux --sort=-%cpu, top/htop, kill, pgrep/pkill
Services:   systemctl status/restart/enable --now, journalctl -u svc -e
Packages:   apt update/install/upgrade (dnf on RHEL)
Network:    ip a, getent/dig, ping, curl -v, ss -ltnp, lsof -i :PORT
Storage:    df -h, df -i, du -sh */, lsblk, truncate -s 0
Health:     uptime/nproc, free -h, dmesg | grep oom
Scripting:  set -euo pipefail, $1 "$@" ${x:-def}, bash -x
Cron:       crontab -e/-l, 5 fields, absolute paths + >> log 2>&1
Security:   ssh-keygen, ufw allow, sudo -l, chmod 600 key
```

## The Incident Loop

```
1. Note symptom + time
2. Resources: top / free -h / df -h
3. Logs around that time: journalctl --since / /var/log
4. Hypothesis -> test -> fix
5. Verify -> Prevent (rotation, monitoring, limits, automation)
```

## Network Debug Order

```
DNS (getent/dig) -> Reachability (ping) -> Port (ss/curl) -> App (curl -v/logs)
- Could not resolve = DNS
- Connection refused = no listener / wrong bind
- Connection timed out = firewall / route
- Address already in use = port conflict
```

## Disk-Full Drill

```
df -h            (which filesystem)
du -sh /path/*   (which directory) | sort -h
df -i            (inodes?)
lsof | grep deleted   (deleted-but-open?)
Fix: truncate -s 0 logs | journalctl --vacuum-size | apt clean | restart holder
```

## Service Debug Drill

```
systemctl status svc -> journalctl -u svc -e -> validate config (-t)
-> check port (ss -ltnp) & perms -> fix -> daemon-reload (if unit changed) -> restart -> verify
```

## Security Essentials

- SSH keys only; disable root + password login.
- Firewall default-deny; open only 22/80/443.
- Patch regularly + auto security updates.
- Least privilege; services as non-root; secrets `600`.
- Back up off-host; test restores.

## Scripting Safety

- Start with `#!/bin/bash` + `set -euo pipefail`.
- Quote variables `"$var"`; validate inputs; provide defaults.
- Never `rm -rf` an unvalidated variable.
- Debug with `bash -x`; lint with `shellcheck`.

## Cron Gotchas

- Minimal environment/PATH; use **absolute paths**.
- Always `>> /path/log 2>&1`.
- Reproduce failures with `env -i bash -c '<cmd>'`.

## DevOps Connections

- **AWS EC2:** Linux server over SSH; security groups + IAM + cloud-init + EBS.
- **Docker:** container = isolated Linux process (namespaces + cgroups).
- **Kubernetes:** nodes are Linux; pods are containers; node issues are Linux issues.
- **CI/CD:** pipeline steps are bash on Linux runners; exit codes decide pass/fail.

## Final Tips

- Always explain the *why*, not just the command.
- Structure beats memorization — use the loops/drills above.
- Reference the labs and projects you built here as real experience.

> You've covered Modules 00–16. Revisit any module's **Quick Recap** for a deeper refresh. Good luck! 🐧
