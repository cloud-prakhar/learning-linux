# Module 05 — Processes & Services

## What You Will Learn

- What a process is and how it lives and dies.
- Viewing processes with `ps`, `top`, and `htop`.
- Signals and killing processes (`kill`, `kill -9`).
- Managing services with `systemd` / `systemctl`.
- Troubleshooting a service that won't start.

## Why This Module Matters

Everything running on Linux is a process. When an app hangs, eats CPU, or won't start, you need to find it, inspect it, and control it. This is core operations work.

## Real-World Use Case

A server is slow. You find the runaway process with `top`, kill it, then ensure the service restarts cleanly with `systemctl` — a classic incident response.

## Topics Covered

| File | What It Covers |
|------|----------------|
| [process-basics.md](./process-basics.md) | PID, parent/child, states |
| [ps-top-htop.md](./ps-top-htop.md) | Viewing & monitoring processes |
| [kill-signals.md](./kill-signals.md) | Signals, kill, killall, pkill |
| [systemd-services.md](./systemd-services.md) | systemctl start/stop/enable |
| [service-troubleshooting.md](./service-troubleshooting.md) | Fixing failed services |

## Learning Flow

```mermaid
flowchart LR
    A[Process basics] --> B[ps/top/htop]
    B --> C[Signals & kill]
    C --> D[systemd services]
    D --> E[Troubleshooting]
```

## Hands-On Practice

Start a background process, find it with `ps`/`top`, send it signals, then inspect and restart a real service like `cron` or `nginx`.

## Common Mistakes

- Using `kill -9` first instead of a graceful `kill`.
- Confusing a one-off process with a managed service.

## Troubleshooting

- Service won't start → `systemctl status` + `journalctl -u` reveal why.
- High CPU → `top`/`htop` to find the offender.

## Best Practices

- Always try graceful termination before `kill -9`.
- Use `systemctl enable` so services survive reboots.

## Quick Revision

- A process has a PID; signals control it; `kill` sends signals.
- `systemd` manages long-running services via `systemctl`.

## Next Module

➡️ [06 — Package Management](../06-package-management/).
