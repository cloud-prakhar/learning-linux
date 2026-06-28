# Module 04 — Users, Groups & Permissions

## What You Will Learn

- How Linux users and groups work.
- How file permissions (read/write/execute) are structured.
- Changing permissions and ownership with `chmod`, `chown`, `chgrp`.
- `sudo` and the root user.
- How to troubleshoot "Permission denied" errors.

## Why This Module Matters

Linux is multi-user and security-focused. Permissions decide who can read, change, or run anything. "Permission denied" is one of the most common beginner blockers — this module makes it obvious.

## Real-World Use Case

You'll create accounts for teammates, lock down a private key, give a script execute permission, and fix permission errors on a web server's files — all standard admin tasks.

## Topics Covered

| File | What It Covers |
|------|----------------|
| [users-and-groups.md](./users-and-groups.md) | Accounts, /etc/passwd, useradd |
| [file-permissions.md](./file-permissions.md) | rwx, owner/group/other |
| [chmod-chown-chgrp.md](./chmod-chown-chgrp.md) | Change permissions & ownership |
| [sudo-and-root.md](./sudo-and-root.md) | Admin power safely |
| [permission-troubleshooting.md](./permission-troubleshooting.md) | Fixing denials |

## Learning Flow

```mermaid
flowchart LR
    A[Users & Groups] --> B[Permissions model]
    B --> C[chmod/chown/chgrp]
    C --> D[sudo & root]
    D --> E[Troubleshooting]
```

## Hands-On Practice

Create a user and group, set file ownership and permissions, then deliberately cause and fix a "Permission denied" error.

## Common Mistakes

- `chmod 777` everything to "make it work" — a serious security risk.
- Running everything as root instead of using `sudo` for specific commands.

## Troubleshooting

- "Permission denied" → check `ls -l`, ownership, and whether you need `sudo`.
- "Operation not permitted" → usually needs root/sudo.

## Best Practices

- Grant the **least privilege** needed.
- Use groups to share access instead of opening files to everyone.

## Quick Revision

- Every file has owner/group/other with read/write/execute bits.
- `chmod` changes permissions; `chown` changes ownership.
- `sudo` runs one command as root; avoid living as root.

## Next Module

➡️ [05 — Processes & Services](../05-processes-and-services/).
