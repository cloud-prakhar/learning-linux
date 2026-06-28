# Cron Troubleshooting

## 1. What Is This?

Diagnosing the classic problem: **"My cron job isn't running"** (even though the script works when you run it by hand).

## 2. Why Is This Needed?

Cron failures are silent and confusing because cron runs in a different environment than your interactive shell. Knowing the usual causes saves hours.

## 3. Simple Layman Explanation

Your script works when *you* run it, but the cron robot runs it in a bare room with different tools and no map. Most "it won't run" issues are about that difference: wrong paths, missing environment, or no record of what happened.

## 4. Technical Explanation

The four usual suspects:
1. **Environment/PATH** — cron has a minimal PATH; your script may rely on commands it can't find.
2. **Relative paths** — cron's working directory isn't where you think.
3. **No output captured** — failures are invisible without `>> log 2>&1`.
4. **Permissions / daemon** — the cron daemon is off, or the user can't access the files.

---

## Scenario: Cron Job Not Running

### Problem
A scheduled job never executes (or executes but does nothing).

### Symptoms
Expected files/log entries never appear; the script works when run manually.

### Possible Causes
Minimal PATH, relative paths, missing output redirection, syntax error, daemon down, wrong user, timezone.

### Commands to Check
```bash
systemctl status cron            # daemon running? (crond on RHEL)
crontab -l                       # is the job actually installed?
grep CRON /var/log/syslog        # cron activity (Debian/Ubuntu)
journalctl -u cron --since "1 hour ago"   # cron logs (systemd)
timedatectl                      # server timezone (schedules use it)
```

### Step-by-Step Fix
1. Confirm the daemon: `systemctl status cron`.
2. Confirm the job exists: `crontab -l`.
3. Add output capture: append `>> /tmp/job.log 2>&1` and inspect it.
4. Use **absolute paths** for the script and any commands it calls.
5. Set environment in the crontab if needed:
   ```cron
   SHELL=/bin/bash
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   0 2 * * * /opt/scripts/backup.sh /etc /backups >> /var/log/backup.log 2>&1
   ```
6. Check `/var/log/syslog` (or `journalctl -u cron`) for `CRON` lines confirming it fired.
7. Verify the running user can read inputs and write outputs.

### Prevention
Always: absolute paths, output logging, manual test, documented schedule.

## 7. Commands (toolbox)

```bash
crontab -l                                  # list jobs
grep CRON /var/log/syslog | tail            # did cron run it?
journalctl -u cron --since today            # cron logs (systemd)
env -i /bin/bash -c '/opt/scripts/job.sh'   # simulate cron's clean env
timedatectl                                 # confirm timezone
```

## 8. Command Explanation

- `grep CRON /var/log/syslog` → shows each time cron launched a job (proves it fired vs the job itself failing).
- `journalctl -u cron` → systemd's view of cron activity.
- `env -i bash -c '...'` → runs your script with an **empty environment**, mimicking cron — the best way to reproduce "works for me but not in cron".
- `timedatectl` → schedules follow the system timezone; a surprising run time is often a TZ mismatch.

## 9. Practice Tasks

1. Schedule `* * * * * env >> /tmp/cronenv.log 2>&1`; after a minute, compare `/tmp/cronenv.log` to your interactive `env` — note the PATH difference.
2. Reproduce a failure with `env -i bash -c '/path/script.sh'`.
3. Find your job's launch in `grep CRON /var/log/syslog`.
4. Add a `PATH=` line to your crontab and re-test a job that previously failed.

## 10. Common Mistakes

- Blaming the schedule when the script fails due to PATH/environment.
- No `>> log 2>&1`, so there's nothing to diagnose.
- Relative paths and assuming the home directory as the working dir.
- Forgetting the timezone affects when jobs run.

## 11. Troubleshooting

This file is the troubleshooting reference. The fastest test is `env -i bash -c '<your command>'` — if it fails there, it'll fail in cron, and the error tells you why.

## 12. Best Practices

- Absolute paths everywhere; set `PATH=`/`SHELL=` in the crontab.
- Always capture output to a log.
- Test under a clean environment before scheduling.
- Keep crontabs and scripts in version control.

## 13. Quick Recap

- Most failures = PATH/environment, relative paths, or no logging.
- Check daemon, `crontab -l`, and cron logs.
- Reproduce with `env -i bash -c '...'`; fix with absolute paths + `PATH=`.

## 14. References

- `man cron`, `man crontab`, `man 5 crontab`
- [crontab-basics.md](./crontab-basics.md)
