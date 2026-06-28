# Module 09 — Logs, Monitoring & Troubleshooting

## What You Will Learn

- Where Linux logs live and how they're organized.
- Reading systemd logs with `journalctl`.
- Traditional logs in `/var/log` and syslog.
- Checking CPU, memory, and disk health.
- Working through real troubleshooting scenarios.

## Why This Module Matters

When something breaks, logs tell you why. Combined with resource checks, they turn "the server is broken" into a precise diagnosis. This is the heart of operations and SRE work.

## Real-World Use Case

An app is throwing errors. You tail its journal, correlate with a memory spike from `free`/`top`, find an out-of-memory kill in the logs, and fix the root cause.

## Topics Covered

| File | What It Covers |
|------|----------------|
| [linux-logs-overview.md](./linux-logs-overview.md) | The logging landscape |
| [journalctl-basics.md](./journalctl-basics.md) | systemd journal queries |
| [syslog-and-var-log.md](./syslog-and-var-log.md) | Classic log files |
| [cpu-memory-disk-checks.md](./cpu-memory-disk-checks.md) | Resource health |
| [real-world-troubleshooting-scenarios.md](./real-world-troubleshooting-scenarios.md) | Putting it together |

## Learning Flow

```mermaid
flowchart LR
    A[Logs overview] --> B[journalctl]
    A --> C[/var/log & syslog]
    B & C --> D[Resource checks]
    D --> E[Real scenarios]
```

## Hands-On Practice

Query logs for a service, filter by time and priority, then correlate an event with CPU/memory/disk readings.

## Common Mistakes

- Reading logs without filtering by time/service → drowning in noise.
- Looking only at the app, ignoring system resources (or vice versa).

## Troubleshooting

- Start from the symptom, find the time window, filter logs to it, then correlate with resources.

## Best Practices

- Filter aggressively (`-u`, `--since`, `-p`).
- Correlate logs with metrics; note timestamps.

## Quick Revision

- `journalctl -u svc -e` for service logs; `/var/log` for classic logs.
- `top`/`free`/`df` for resources.
- Diagnose by correlating logs + metrics around the incident time.

## Next Module

➡️ [10 — Shell Scripting](../10-shell-scripting/).
