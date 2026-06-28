# Learning Linux

> A structured, beginner-friendly path to learn Linux from **zero to intermediate** — built for people heading into **DevOps, Cloud, SRE, System Administration, and IT Operations**.

Linux runs most of the internet: web servers, cloud instances (AWS/Azure/GCP), Docker containers, Kubernetes nodes, CI/CD agents, and the majority of production systems. If you want a career in modern IT, Linux is not optional — it is the foundation everything else sits on.

This repository teaches Linux the way a good mentor would: **concept → analogy → real-world example → diagram → hands-on commands → troubleshooting → practice.**

---

## Who This Repo Is For

- Complete beginners who have never used a terminal.
- People coming from **Windows / macOS** with no Linux background.
- Aspiring **DevOps / Cloud / SRE / SysAdmin / IT Ops** engineers.
- Students and self-learners who want a clear, sequential path.
- Anyone who wants real, practical, hands-on Linux skills — not just theory.

**No prior Linux knowledge is required.** Basic computer literacy is enough.

---

## Why Linux Matters

| Where | How Linux Shows Up |
|-------|--------------------|
| Cloud | Most AWS EC2 / Azure / GCP instances run Linux |
| Containers | Docker images are built on Linux base images |
| Kubernetes | Worker nodes are Linux machines |
| Web | Nginx, Apache, and most web servers run on Linux |
| DevOps | CI/CD runners, build agents, and automation run on Linux |
| Servers | The majority of production servers worldwide are Linux |

Learn Linux once, and it pays off across **every** layer of modern infrastructure.

---

## Learning Path (Beginner → Intermediate)

| Level | Modules | You Will Be Able To |
|-------|---------|---------------------|
| 🟢 Foundation | 00–02 | Understand what Linux is, set it up, and navigate the system |
| 🟢 Core Skills | 03–06 | Manage files, users, permissions, processes, and packages |
| 🟡 Operations | 07–09 | Handle networking, storage, logs, and troubleshooting |
| 🟡 Automation | 10–11 | Write shell scripts and schedule jobs with cron |
| 🟠 Production | 12–13 | Apply security basics and connect Linux to DevOps/Cloud |
| 🟠 Mastery | 14–17 | Practice labs, build projects, revise, and prep for interviews |

---

## Module Roadmap

| # | Module | Focus |
|---|--------|-------|
| 00 | [Getting Started](./00-getting-started/) | What & why of Linux, real-world usage, roadmap |
| 01 | [Linux Setup](./01-linux-setup/) | WSL, VirtualBox, cloud servers, terminal basics |
| 02 | [Linux Basics](./02-linux-basics/) | Architecture, kernel/shell, filesystem, navigation |
| 03 | [Files & Directories](./03-files-and-directories/) | Create, view, search, links, compression |
| 04 | [Users, Groups & Permissions](./04-users-groups-permissions/) | Users, permissions, chmod/chown, sudo |
| 05 | [Processes & Services](./05-processes-and-services/) | ps/top, signals, systemd services |
| 06 | [Package Management](./06-package-management/) | apt, yum/dnf, install/remove/update |
| 07 | [Networking Basics](./07-networking-basics/) | IP/DNS, ping/curl, ports, ss/lsof |
| 08 | [Storage & Disk](./08-storage-and-disk-management/) | Partitions, mount, df/du, disk-full fixes |
| 09 | [Logs, Monitoring & Troubleshooting](./09-logs-monitoring-troubleshooting/) | journalctl, /var/log, resource checks |
| 10 | [Shell Scripting](./10-shell-scripting/) | Variables, loops, functions, real scripts |
| 11 | [Automation & Cron](./11-automation-and-cron/) | Scheduling jobs, backups, debugging cron |
| 12 | [Security Basics](./12-linux-security-basics/) | SSH, firewall, least privilege |
| 13 | [Linux for DevOps](./13-real-world-linux-for-devops/) | AWS, Docker, Kubernetes, CI/CD |
| 14 | [Hands-On Labs](./14-hands-on-labs/) | Guided practical exercises |
| 15 | [Mini Projects](./15-mini-projects/) | Build real, useful scripts & setups |
| 16 | [Cheatsheets](./16-cheatsheets/) | Quick-reference command tables |
| 17 | [Interview & Revision](./17-interview-and-revision/) | Q&A, scenarios, final notes |

Plus: [diagrams/](./diagrams/) (visual reference) and [references/](./references/) (official docs).

---

## How To Use This Repo

1. **Go in order.** Modules build on each other — start at `00` and move forward.
2. **Read, then do.** Every topic ends with **Practice Tasks** — actually run them.
3. **Break things safely.** Use a throwaway VM/WSL/cloud instance to experiment.
4. **Keep cheatsheets open** ([module 16](./16-cheatsheets/)) while practicing.
5. **Test yourself** with [module 17](./17-interview-and-revision/) when you finish a section.

> 💡 The fastest way to learn Linux is to **type every command yourself**. Reading is not enough.

---

## Prerequisites

- A computer (Windows, macOS, or Linux).
- Internet connection.
- Willingness to use the terminal and make mistakes.
- ~30–60 minutes a day. Consistency beats cramming.

---

## Suggested Practice Environment

Pick **one** to start (covered in [Module 01](./01-linux-setup/)):

| Option | Best For | Difficulty |
|--------|----------|------------|
| **WSL** (Windows Subsystem for Linux) | Windows users, fastest start | ⭐ Easy |
| **VirtualBox + Ubuntu** | Full desktop Linux experience | ⭐⭐ Medium |
| **Cloud server** (AWS EC2 free tier) | Real-world server feel | ⭐⭐ Medium |

---

## Hands-On Labs

Guided, scenario-based exercises in [Module 14](./14-hands-on-labs/):
basic commands, permissions, process/service debugging, network debugging, a disk-full scenario, and shell automation.

## Mini Projects

Build something real in [Module 15](./15-mini-projects/):
a health-check script, log-cleanup automation, a user-management script, an Nginx setup, and a troubleshooting playbook.

## Cheatsheets

Fast lookups in [Module 16](./16-cheatsheets/): basic commands, permissions, networking, processes/services, and troubleshooting.

## Interview Preparation

[Module 17](./17-interview-and-revision/) covers beginner & intermediate interview questions, scenario-based questions, command practice, and final revision notes — tuned for DevOps/Cloud/SRE/IT Ops roles.

---

## Contribution Guidelines

Contributions are welcome!

1. Fork the repo and create a branch: `git checkout -b improve-module-07`.
2. Follow the existing **topic file template** (14 sections) and **module README template**.
3. Keep language beginner-friendly; explain jargon; add a diagram where it helps.
4. No destructive commands. Scripts must be safe and commented.
5. Verify any links point to **real official documentation**.
6. Open a Pull Request with a clear description of what you added/changed.

---

## Official References

The [references/](./references/) folder collects the authoritative, official sources:

| File | What's Inside |
|------|---------------|
| [official-documentation.md](./references/official-documentation.md) | man pages, Ubuntu/Debian/Red Hat docs, GNU Coreutils, Bash, systemd, OpenSSH, FHS |
| [standards-and-rfcs.md](./references/standards-and-rfcs.md) | The specs that define it all: FHS, POSIX, and IETF RFCs (TCP/IP, DNS, HTTP, SSH, TLS) |
| [reading-list-by-module.md](./references/reading-list-by-module.md) | Per-module official docs + `man` pages to read alongside each module |
| [recommended-learning-resources.md](./references/recommended-learning-resources.md) | Trusted tutorials, practice sites, and books |

> **Reading order that never lies:** `man <command>` (offline, version-exact) → official docs (context) → RFCs/standards (ground truth).

---

## License & Usage

Free to use for self-learning and teaching. Star ⭐ the repo if it helps you, and share it with someone starting their Linux journey.

> **Start here → [00-getting-started](./00-getting-started/)**

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| — | 🏠 Home | ➡️ Next: [Module 00 — Getting Started](00-getting-started/README.md) |
