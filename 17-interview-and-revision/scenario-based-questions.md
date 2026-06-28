# Scenario-Based Questions

> These test *how you think*. Narrate a structured approach — interviewers care about method as much as the answer.

## Scenario 1: "The website is down."

**Approach (debug in layers):**
1. Is it DNS? `getent hosts site.com` / `dig`.
2. Is the host reachable? `ping <ip>`.
3. Is the service listening? `ss -ltnp | grep :443`; `systemctl status nginx`.
4. Does it respond? `curl -v https://localhost`.
5. Read logs: `journalctl -u nginx -e` / `/var/log/nginx/error.log`.
6. Check resources/disk: `df -h`, `free -h`.

**Key point:** isolate the failing layer (DNS → reachability → port → app) before changing anything.

## Scenario 2: "The server is slow."

**Approach:**
1. `uptime` vs `nproc` — is load high?
2. `top`/`htop` — CPU hog? sort by CPU (P) and memory (M).
3. `free -h` — swapping?
4. `iostat -xz 1 3` / `top` `%wa` — disk I/O wait?
5. Act: kill/限 a runaway process, free memory, reduce I/O.

**Key point:** identify the bottleneck (CPU vs memory vs I/O) — don't guess.

## Scenario 3: "Disk is full and the app stopped writing."

**Approach:**
1. `df -h` — which filesystem?
2. `du -sh /var/* | sort -h` — drill to the biggest dir.
3. `df -i` — inodes exhausted?
4. `lsof | grep deleted` — space held by deleted-open files?
5. Reclaim safely: `truncate -s 0` active logs, `journalctl --vacuum-size`, `apt clean`; restart holders.
6. Prevent: logrotate + monitoring.

**Key point:** mention the deleted-but-open trap and safe truncation — it shows real experience.

## Scenario 4: "A new deploy gets 'Permission denied' reading files."

**Approach:**
1. `id` (which user runs the deploy) and `ls -l` (who owns the files).
2. `namei -l /path` — find which directory blocks access.
3. Fix with correct ownership/group: `sudo chown -R deploy:deploy /app` and proper perms (dirs 755, files 644).

**Key point:** diagnose owner/perms/parent-traversal before applying the minimal fix (never `777`).

## Scenario 5: "Cron job isn't running."

**Approach:**
1. `crontab -l` — is it installed?
2. `systemctl status cron` — daemon up?
3. `grep CRON /var/log/syslog` — did it fire?
4. Add `>> log 2>&1`; reproduce with `env -i bash -c '<command>'` to mimic cron's environment.
5. Fix: absolute paths, `PATH=`/`SHELL=` in crontab.

**Key point:** the usual cause is environment/PATH, not the schedule.

## Scenario 6: "A systemd service keeps crashing (CrashLoopBackOff in K8s)."

**Approach:**
1. `systemctl status svc` / `kubectl logs <pod>` — read the exit error.
2. `journalctl -u svc -e` — root cause.
3. Check OOM (`dmesg | grep -i oom`), config errors, missing deps, disk.
4. Fix root cause; add `Restart=on-failure` / resource limits.

**Key point:** a crashing container/service is a Linux process exiting — read its logs and exit code.

## Scenario 7: "You're handed a new production server. What do you check before go-live?"

**Approach (checklist):**
SSH keys + no root/password login → firewall default-deny (22/80/443) → patched + auto security updates → least privilege → services enabled → disk/monitoring → log rotation → backups (tested) → time sync → runbook. (See [production checklist](../13-real-world-linux-for-devops/production-server-checklist.md).)

## Scenario 8: "How do you find what changed recently when something broke?"

**Approach:**
- `ls -lt /etc` and `journalctl --since "2 hours ago"` around the incident time.
- Check package history (`/var/log/dpkg.log`), recent logins (`last`), and `journalctl -b -1` (previous boot).
- Correlate the change time with the failure time.

**Key point:** anchor everything to the **incident timestamp**.

## How to Practice

- Say your steps out loud; structure beats memorization.
- Always end with **prevention** (monitoring, rotation, limits, automation).
- Tie answers to the labs/projects you built in this repo.

> Source: Modules 05, 07, 08, 09, 11, 12, 13.
