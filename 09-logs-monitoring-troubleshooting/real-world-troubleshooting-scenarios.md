# Real-World Troubleshooting Scenarios

## 1. What Is This?

End-to-end walkthroughs that combine logs + resource checks + service tools to solve realistic incidents.

## 2. Why Is This Needed?

Individual commands are easy; **combining** them under pressure is the real skill. These scenarios build that workflow.

## 3. Simple Layman Explanation

It's the difference between knowing how to use a stethoscope and actually diagnosing a patient. Here you practice full diagnoses from symptom to fix.

## 4. Technical Explanation

General method (the "incident loop"):
1. Reproduce / note the **symptom** and **time**.
2. Check **resources** (`top`, `free`, `df`).
3. Check **logs** around that time (`journalctl --since`, `/var/log`).
4. Form a hypothesis → test it → fix → verify → prevent.

---

## Scenario A: Server Is Slow

### Symptoms
Web responses lag; SSH feels sluggish.

### Commands to Check
```bash
uptime                       # load vs cores
top                          # who's using CPU? (P)
free -h                      # memory pressure / swap
iostat -xz 1 3               # disk I/O wait
```

### Step-by-Step Fix
1. `uptime` vs `nproc` → is load high?
2. `top` → identify the top CPU/memory process.
3. If a runaway process → investigate or `kill` it (Module 05).
4. If swapping → reduce memory use / add RAM.
5. If `%iowait` high → reduce disk I/O / check the disk.

### Prevention
Monitoring + alerts; capacity planning.

---

## Scenario B: A Service Keeps Crashing

### Symptoms
`systemctl status app` shows repeated restarts/failures.

### Commands to Check
```bash
systemctl status app
journalctl -u app --since "30 min ago" -e
dmesg | grep -i oom
df -h
```

### Step-by-Step Fix
1. Read the exact error in `journalctl`.
2. If OOM-killed → memory issue (Scenario A).
3. If config error → fix config, validate, restart.
4. If disk full → free space (Module 08).
5. Restart and confirm `active (running)`.

### Prevention
Set `Restart=on-failure`, memory limits, monitoring.

---

## Scenario C: High CPU From One Process

### Symptoms
`top` shows one process at ~100% CPU.

### Commands to Check
```bash
top                              # note the PID
ps -p <pid> -o pid,ppid,cmd,%cpu,%mem
journalctl -u <service> -e       # if it's a managed service
```

### Step-by-Step Fix
1. Identify the PID and command.
2. Decide: legitimate spike vs runaway/bug.
3. If safe, `kill <pid>` (graceful first) or restart its service.
4. Investigate the root cause in its logs.

### Prevention
Resource limits (cgroups/systemd), code fixes, alerts.

---

## Scenario D: Out of Disk Space

### Symptoms
Writes fail with "No space left on device".

### Commands to Check
```bash
df -h ; df -i
du -sh /var/* 2>/dev/null | sort -h
sudo lsof | grep deleted
```

### Step-by-Step Fix
See [Module 08 disk-full-troubleshooting](../08-storage-and-disk-management/disk-full-troubleshooting.md): locate FS → drill with `du` → clean safely (truncate/journal vacuum/apt clean) → restart processes holding deleted files.

### Prevention
Log rotation, journald limits, disk alerts.

## 7. Commands (toolbox)

```bash
uptime; top; free -h          # resources
df -h; df -i; du -sh */       # disk
systemctl status <svc>        # service state
journalctl -u <svc> --since "X min ago" -e   # service logs
dmesg | grep -i oom           # memory kills
```

## 8. Command Explanation

These are covered in their topic files; here they form the diagnostic loop. The skill is choosing the right one based on the symptom and correlating timestamps.

## 9. Practice Tasks

1. Generate CPU load: `yes > /dev/null &` (then `top`, find it, `kill` it).
2. Inspect a real service: `journalctl -u ssh --since "1 hour ago"`.
3. Run the full resource trio (`uptime`, `free -h`, `df -h`) and write a one-line health summary.
4. Walk through Scenario D using a temp junk file (`fallocate`).

## 10. Common Mistakes

- Jumping to a fix before identifying the bottleneck.
- Ignoring timestamps — not correlating logs with the incident window.
- Looking only at the app or only at resources, not both.

## 11. Troubleshooting

This file *is* applied troubleshooting. When stuck, return to the loop: symptom/time → resources → logs → hypothesis → fix → verify → prevent.

## 12. Best Practices

- Note the incident time first; filter everything to that window.
- Change one thing at a time and re-test.
- Always add a prevention step (the "post-incident" habit).

## 13. Quick Recap

- Incident loop: symptom → resources → logs → fix → verify → prevent.
- Correlate metrics and logs by timestamp.
- Each scenario maps symptoms to specific commands.

## 14. References

- [Module 05](../05-processes-and-services/), [Module 08](../08-storage-and-disk-management/)
- [Module 15 Project 05: troubleshooting playbook](../15-mini-projects/project-05-troubleshooting-playbook.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [CPU, Memory, and Disk Checks](cpu-memory-disk-checks.md) | ⬆️ Module: [Module 09 — Logs, Monitoring & Troubleshooting](README.md) | ➡️ Next: [Module 10 — Shell Scripting](../10-shell-scripting/README.md) |
