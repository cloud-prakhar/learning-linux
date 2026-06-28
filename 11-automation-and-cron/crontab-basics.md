# Crontab Basics

## 1. What Is This?

The **crontab syntax**: five time fields plus a command, defining *when* and *what* cron runs.

## 2. Why Is This Needed?

To schedule anything you must write the schedule correctly. Misreading the five fields is the most common cron mistake.

## 3. Simple Layman Explanation

A crontab line is a sentence: "**At this minute, this hour, this day, this month, this weekday — run this command.**" Get the five time words right and cron does the rest.

## 4. Technical Explanation

```
* * * * *  command_to_run
| | | | |
| | | | +-- day of week (0-7, 0 and 7 = Sunday)
| | | +---- month (1-12)
| | +------ day of month (1-31)
| +-------- hour (0-23)
+---------- minute (0-59)
```

Special operators:
- `*` = every value
- `*/5` = every 5 (e.g., every 5 minutes)
- `1,15` = at 1 and 15
- `9-17` = range 9 through 17

Shortcuts: `@reboot`, `@hourly`, `@daily`, `@weekly`, `@monthly`.

## 5. Real-World Example

| Schedule | Meaning |
|----------|---------|
| `0 2 * * *` | Every day at 02:00 (nightly backup) |
| `*/5 * * * *` | Every 5 minutes (health check) |
| `0 9 * * 1` | Every Monday at 09:00 (weekly report) |
| `30 0 1 * *` | 00:30 on the 1st of each month |
| `@reboot` | Once at every boot |

## 6. Diagram

```mermaid
flowchart LR
    M[minute] --> H[hour] --> Dom[day-of-month] --> Mon[month] --> Dow[day-of-week] --> C[command]
```

## 7. Commands

```bash
crontab -e        # edit your crontab
crontab -l        # list current jobs
```

Example crontab content:

```cron
# Run a backup every day at 2 AM (use absolute paths!)
0 2 * * * /home/alice/scripts/backup.sh /etc /backups >> /home/alice/logs/backup.log 2>&1

# Health check every 5 minutes
*/5 * * * * /home/alice/scripts/healthcheck.sh >> /home/alice/logs/health.log 2>&1

# Weekly cleanup, Sundays at 3:30 AM
30 3 * * 0 /home/alice/scripts/cleanup-logs.sh /var/log/myapp 7 30 >> /home/alice/logs/cleanup.log 2>&1
```

## 8. Command Explanation

- The five fields set the schedule; the rest is the exact command.
- `>> file 2>&1` → append both **stdout** and **stderr** to a log. **Always do this** — otherwise cron may email output or you'll never see errors.
- Use **absolute paths** for scripts and binaries (cron's PATH is minimal).
- Lines starting with `#` are comments; document each job.

## 9. Practice Tasks

1. Add: `* * * * * date >> /tmp/cron-test.log 2>&1`, wait 2 minutes, check the file grows.
2. Change it to `*/2 * * * *` (every 2 minutes) and verify the cadence.
3. Write the cron line for "every weekday at 6 PM".
4. Remove the test line with `crontab -e`.

## 10. Common Mistakes

- Confusing field order (minute is **first**, not hour).
- Forgetting `>> log 2>&1`, so failures are invisible.
- Using `%` literally in a command (in crontab, `%` is special — escape it as `\%`).
- Relative paths and assuming your normal PATH.

## 11. Troubleshooting

- **Job runs at the wrong time** → re-read the five fields; check the server timezone (`timedatectl`).
- **No output** → add `>> /tmp/job.log 2>&1` and inspect it.
- **Validate your schedule** with a crontab expression checker before trusting it.

## 12. Best Practices

- Always redirect output to a log.
- Use absolute paths; set `PATH=` at the top of the crontab if needed.
- Comment every job with its purpose.
- Test the script manually first, then schedule it.

## 13. Quick Recap

- Five fields: minute hour day-of-month month day-of-week.
- `*/5` every five; `0 2 * * *` daily at 2 AM.
- Absolute paths + `>> log 2>&1` always.

## 14. References

- `man 5 crontab`
- crontab.guru (expression helper): https://crontab.guru/
