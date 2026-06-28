# Lab 03 — Process & Service Debugging

## Objective

Find and control processes (including a CPU hog), then inspect and manage a real systemd service using `journalctl`.

## Scenario

A server is "slow" and a service needs checking. You'll generate load, identify the culprit, kill it, then practice service inspection — the core of incident response.

## Prerequisites

- A Linux environment.
- Modules 05 and 09 read.

## Steps & Commands

```bash
# 1. Start a harmless CPU-burning background process
yes > /dev/null &
echo "Started PID: $!"          # $! = PID of the last background job

# 2. Find it
top -b -n 1 | head -15          # snapshot; look for 'yes' near the top
ps aux --sort=-%cpu | head -5   # top CPU consumers
pgrep -a yes                    # PID(s) of 'yes'

# 3. Kill it gracefully, then confirm
kill "$(pgrep yes)"
pgrep yes || echo "yes is gone"

# 4. Inspect a real service (cron or ssh)
systemctl status cron 2>/dev/null || systemctl status ssh
systemctl is-active cron 2>/dev/null || systemctl is-active ssh

# 5. Read its recent logs
journalctl -u cron --since "1 hour ago" --no-pager | tail -20 \
  || journalctl -u ssh --since "1 hour ago" --no-pager | tail -20

# 6. Restart and watch the journal update
sudo systemctl restart cron 2>/dev/null || sudo systemctl restart ssh
systemctl status cron 2>/dev/null || systemctl status ssh
```

## Expected Output (samples)

```
$ ps aux --sort=-%cpu | head -3
USER  PID  %CPU ... COMMAND
you   1234 99.0 ... yes

$ pgrep yes || echo "yes is gone"
yes is gone

$ systemctl is-active cron
active
```

## Explanation

- `yes > /dev/null &` → spins the CPU by printing endlessly to the null device; `&` backgrounds it; `$!` captures its PID.
- `ps aux --sort=-%cpu` → ranks by CPU so the hog is at the top (Module 05).
- `pgrep -a yes` → finds the PID by name; `kill` sends SIGTERM (graceful).
- `systemctl status`/`is-active` → service state (Module 05).
- `journalctl -u <svc>` → that service's logs (Module 09); `--no-pager` prints directly.

## Validation

- `pgrep yes` returns nothing after the kill.
- `systemctl is-active` reports `active` for the chosen service after restart.

## Cleanup

```bash
pkill yes 2>/dev/null    # ensure no stray 'yes' remains
```

## Troubleshooting

- **`yes` still running** → `pkill -9 yes` as a last resort.
- **No `cron` service** → use `ssh` (the lab falls back automatically).
- **`journalctl` permission denied** → add `sudo` or join the `systemd-journal` group.
- **`top` floods the screen** → the lab uses `top -b -n 1` (batch, one snapshot); press `q` if you run interactive `top`.

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Lab 02 — User & Permission Practice](lab-02-user-permission-practice.md) | ⬆️ Module: [Module 14 — Hands-On Labs](README.md) | ➡️ Next: [Lab 04 — Network Debugging](lab-04-network-debugging.md) |
