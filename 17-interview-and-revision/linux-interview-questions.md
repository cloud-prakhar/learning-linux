# Linux Interview Questions

> Attempt each question first, then read the answer. Tuned for DevOps/Cloud/SRE/IT Ops.

## Beginner

**1. What is Linux and what is the kernel?**
Linux is a free, open-source OS. The **kernel** is its core, managing hardware (CPU, memory, devices) and exposing system calls. A full system (kernel + tools + package manager) is a **distribution** (Ubuntu, RHEL).

**2. Difference between a shell and a terminal?**
The **terminal** is the window/app; the **shell** (e.g., bash) is the program inside it that interprets your commands.

**3. What does `ls -lah` show?**
Long listing (`-l`), including hidden files (`-a`), with human-readable sizes (`-h`): permissions, owner, group, size, date, name.

**4. Absolute vs relative path?**
Absolute starts from `/` and works anywhere (`/var/log`). Relative starts from your current directory (`log`). Scripts should prefer absolute.

**5. How do you view the last 50 lines of a log and follow it live?**
`tail -n 50 file` and `tail -f file`.

**6. What are the three permission types and classes?**
Types: read (4), write (2), execute (1). Classes: owner, group, others. E.g., `rwxr-xr--` = 754.

**7. How do you make a script executable and run it?**
`chmod +x script.sh` then `./script.sh` (or `bash script.sh`).

**8. What is `sudo`?**
Runs a single command with root privileges, checked against `/etc/sudoers` and logged — safer than logging in as root.

**9. How do you check disk space and memory?**
`df -h` (disk), `free -h` (memory), `du -sh dir` (directory size).

**10. How do you install a package on Ubuntu vs RHEL?**
`sudo apt install pkg` (Ubuntu/Debian) vs `sudo dnf install pkg` (RHEL/Fedora). On apt, run `apt update` first.

## Intermediate

**11. Difference between `kill`, `kill -9`, and `kill -HUP`?**
`kill` (SIGTERM 15) asks a process to exit gracefully; `kill -9` (SIGKILL) forces it with no cleanup (last resort); `kill -HUP` often reloads config.

**12. How does systemd differ between `enable` and `start`?**
`start` runs the service now; `enable` makes it start at boot. They're independent — use `enable --now` for both.

**13. A service won't start. Walk me through debugging.**
`systemctl status svc` → `journalctl -u svc -e` for the error → validate config (`nginx -t`) → check port conflicts (`ss -ltnp`) and permissions → fix → restart → verify.

**14. Disk shows free space but writes fail. Why?**
Inodes are exhausted (`df -i`) — too many small files. Or a deleted-but-open file is holding space (`lsof | grep deleted`); restart the holder.

**15. Difference between a hard link and a symlink?**
A hard link is another name for the same inode (same data). A symlink is a pointer to a path; it breaks if the target is removed and can cross filesystems/link directories.

**16. How do you debug "connection refused" vs "timed out"?**
Refused = something reachable actively rejected you (no listener/wrong bind) — check `ss -ltnp`, the service. Timed out = firewall/route/host down — check firewall/security group and `ping`.

**17. What's the difference between `apt update` and `apt upgrade`?**
`update` refreshes the package index (list of available versions); `upgrade` installs newer versions of installed packages.

**18. How do you find which process uses port 80?**
`sudo ss -ltnp | grep :80` or `sudo lsof -i :80`.

**19. Why might a cron job work manually but not in cron?**
Cron runs with a minimal environment/PATH and a different working directory. Use absolute paths, set `PATH=` in the crontab, and redirect output to a log.

**20. How do you harden SSH on a server?**
Use key-based auth, set `PermitRootLogin no` and `PasswordAuthentication no` in `sshd_config`, restrict by firewall/IP, and reload sshd (keeping a session open to test).

**21. What is least privilege and how do you apply it?**
Give users/services only the access they need: services run as non-root users (e.g., `www-data`), scoped sudo per command, tight file permissions, secrets `600`.

**22. How do you safely free space from a full disk?**
`df -h` to find the FS, `du -sh /path/* | sort -h` to find the directory, then `truncate -s 0` active logs, `journalctl --vacuum-size`, `apt clean`; restart processes holding deleted files.

**23. Explain how Docker containers relate to Linux.**
A container is an isolated Linux process using namespaces (isolation) and cgroups (limits), sharing the host kernel — not a full VM. Debug it like a Linux process (logs, exit code, ports, perms).

**24. What does `set -euo pipefail` do?**
In bash scripts: exit on any error (`-e`), error on unset variables (`-u`), and fail if any command in a pipeline fails (`pipefail`). A safety header.

**25. How do you check what's consuming CPU/memory right now?**
`top`/`htop` (sort with P/M), or `ps aux --sort=-%cpu | head` / `--sort=-%mem`.

## Tips for Answering

- State the command **and** the reasoning.
- For "how would you debug", describe a **structured approach** (layers/incident loop), not just one command.
- Reference real practice ("in a lab I…").

> Source: all modules. See [scenario-based-questions.md](./scenario-based-questions.md) next.

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 17 — Interview & Revision](README.md) | ⬆️ Module: [Module 17 — Interview & Revision](README.md) | ➡️ Next: [Scenario-Based Questions](scenario-based-questions.md) |
