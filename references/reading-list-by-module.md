# Official Reading List by Module

For each module, this is the **official documentation** and the **`man` pages** to read alongside it. Work through a module here, then read its references to deepen and confirm your understanding from the primary sources.

> **`man` pages are the most reliable reference of all** — they're installed on every Linux system and match *your* exact version. Read them with `man <name>` (press `q` to quit). For a section number, use `man 5 crontab`.

---

## 00 — Getting Started

- The Linux Kernel: https://www.kernel.org/
- Linux Foundation: https://www.linuxfoundation.org/
- **man:** `man intro`, `man uname`, `man hostnamectl`

## 01 — Linux Setup

- WSL (Windows): https://learn.microsoft.com/windows/wsl/
- Ubuntu Desktop install: https://ubuntu.com/tutorials/install-ubuntu-desktop
- VirtualBox: https://www.virtualbox.org/manual/
- AWS EC2: https://docs.aws.amazon.com/ec2/
- **man:** `man bash`, `man ssh`

## 02 — Linux Basics

- Filesystem Hierarchy Standard: https://refspecs.linuxfoundation.org/fhs.shtml
- GNU Bash manual: https://www.gnu.org/software/bash/manual/
- **man:** `man hier`, `man bash`, `man pwd`, `man ls`, `man cd`

## 03 — Files & Directories

- GNU Coreutils: https://www.gnu.org/software/coreutils/manual/
- GNU Tar: https://www.gnu.org/software/tar/manual/
- GNU Findutils: https://www.gnu.org/software/findutils/manual/
- GNU grep: https://www.gnu.org/software/grep/manual/
- **man:** `man cp`, `man mv`, `man rm`, `man find`, `man grep`, `man tar`, `man ln`

## 04 — Users, Groups & Permissions

- Ubuntu user management: https://ubuntu.com/server/docs/security-users
- sudo project: https://www.sudo.ws/docs/
- **man:** `man chmod`, `man chown`, `man useradd`, `man usermod`, `man passwd`, `man sudo`, `man sudoers`, `man umask`

## 05 — Processes & Services

- systemd: https://www.freedesktop.org/wiki/Software/systemd/
- systemd man pages: https://www.freedesktop.org/software/systemd/man/
- **man:** `man ps`, `man top`, `man kill`, `man 7 signal`, `man systemctl`, `man systemd.service`

## 06 — Package Management

- Ubuntu/Debian APT: https://ubuntu.com/server/docs/package-management
- Debian packaging reference: https://www.debian.org/doc/manuals/debian-reference/ch02.en.html
- DNF: https://dnf.readthedocs.io/
- **man:** `man apt`, `man dpkg`, `man dnf`, `man rpm`

## 07 — Networking Basics

- iproute2: https://wiki.linuxfoundation.org/networking/iproute2
- curl docs: https://curl.se/docs/
- See also: [standards-and-rfcs.md](./standards-and-rfcs.md) (IP, TCP, UDP, DNS, HTTP)
- **man:** `man ip`, `man ss`, `man dig`, `man curl`, `man ping`, `man lsof`, `man resolv.conf`

## 08 — Storage & Disk Management

- Ubuntu storage docs: https://ubuntu.com/server/docs
- **man:** `man lsblk`, `man df`, `man du`, `man mount`, `man umount`, `man fstab`, `man blkid`, `man logrotate`

## 09 — Logs, Monitoring & Troubleshooting

- systemd journal: https://www.freedesktop.org/software/systemd/man/journalctl.html
- rsyslog: https://www.rsyslog.com/doc/
- **man:** `man journalctl`, `man dmesg`, `man rsyslog.conf`, `man top`, `man free`, `man vmstat`, `man iostat`

## 10 — Shell Scripting

- GNU Bash manual: https://www.gnu.org/software/bash/manual/
- ShellCheck (linter): https://www.shellcheck.net/
- POSIX shell: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html
- **man:** `man bash`, `man test`, `man set`

## 11 — Automation & Cron

- Ubuntu cron howto: https://help.ubuntu.com/community/CronHowto
- systemd timers (alternative): https://www.freedesktop.org/software/systemd/man/systemd.timer.html
- **man:** `man cron`, `man crontab`, `man 5 crontab`

## 12 — Linux Security Basics

- OpenSSH manual: https://www.openssh.com/manual.html
- ufw: https://help.ubuntu.com/community/UFW
- firewalld: https://firewalld.org/documentation/
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks
- See also: [standards-and-rfcs.md](./standards-and-rfcs.md) (SSH, TLS)
- **man:** `man ssh`, `man ssh-keygen`, `man sshd_config`, `man ufw`, `man firewall-cmd`

## 13 — Real-World Linux for DevOps

- AWS EC2: https://docs.aws.amazon.com/ec2/
- cloud-init: https://cloudinit.readthedocs.io/
- Docker: https://docs.docker.com/
- Kubernetes: https://kubernetes.io/docs/
- Linux namespaces: https://man7.org/linux/man-pages/man7/namespaces.7.html
- **man:** `man namespaces`, `man cgroups`

## 14–15 — Labs & Mini Projects

- Reuse the docs from the modules each lab/project draws on (listed in their headers).
- ShellCheck for scripts: https://www.shellcheck.net/
- explainshell (decode any command): https://explainshell.com/

## 16–17 — Cheatsheets, Interview & Revision

- kubectl cheatsheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- Linux Foundation certifications (LFCS): https://training.linuxfoundation.org/
- Red Hat RHCSA: https://www.redhat.com/en/services/training-and-certification

---

## A Reliable Reading Workflow

1. **Do the module** in this repo (concept → practice).
2. **Read the man pages** for its commands — they match your system exactly.
3. **Skim the official docs** above for the bigger picture.
4. **Hit the RFC/standard** ([standards-and-rfcs.md](./standards-and-rfcs.md)) only when you need the precise, authoritative behavior.

> man pages first (offline, version-exact) → official docs (context) → RFCs/standards (ground truth).

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Standards & RFCs](standards-and-rfcs.md) | ⬆️ Module: [References](README.md) | ➡️ Next: [Recommended Learning Resources](recommended-learning-resources.md) |
