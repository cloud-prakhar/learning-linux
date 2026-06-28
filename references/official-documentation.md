# Official Documentation

The authoritative sources. Prefer these over third-party content.

## Built-in (Always Available)

| Resource | How to Access |
|----------|---------------|
| Manual pages | `man <command>` (e.g., `man ls`); `q` to quit |
| Section search | `man -k <keyword>` / `apropos <keyword>` |
| Quick usage | `<command> --help` |
| Info pages | `info <command>` (GNU tools) |
| Filesystem layout | `man hier` |

> The `man` pages are the single most reliable reference — installed on every Linux system, even offline.

## The Kernel

- The Linux Kernel: https://www.kernel.org/
- Kernel documentation: https://docs.kernel.org/

## Distributions

| Distro | Documentation |
|--------|---------------|
| Ubuntu | https://ubuntu.com/server/docs and https://help.ubuntu.com/ |
| Debian | https://www.debian.org/doc/ |
| Red Hat Enterprise Linux | https://access.redhat.com/documentation/ |
| Fedora | https://docs.fedoraproject.org/ |
| Arch Wiki (excellent reference, broadly useful) | https://wiki.archlinux.org/ |

## Core Tools & Shell

| Tool | Documentation |
|------|---------------|
| GNU Coreutils (ls, cp, mv, cat, etc.) | https://www.gnu.org/software/coreutils/manual/ |
| GNU Bash | https://www.gnu.org/software/bash/manual/ |
| GNU grep | https://www.gnu.org/software/grep/manual/ |
| GNU tar | https://www.gnu.org/software/tar/manual/ |
| GNU findutils | https://www.gnu.org/software/findutils/manual/ |

## System & Services

| Topic | Documentation |
|-------|---------------|
| systemd / systemctl / journalctl | https://www.freedesktop.org/wiki/Software/systemd/ |
| systemd man pages | https://www.freedesktop.org/software/systemd/man/ |
| cron / crontab | `man 5 crontab`; https://help.ubuntu.com/community/CronHowto |
| logrotate | `man logrotate` |
| rsyslog | https://www.rsyslog.com/doc/ |

## Networking

| Topic | Documentation |
|-------|---------------|
| iproute2 (ip, ss) | https://wiki.linuxfoundation.org/networking/iproute2 |
| curl | https://curl.se/docs/ |
| BIND/DNS concepts | https://www.cloudflare.com/learning/dns/what-is-dns/ |
| IANA port registry | https://www.iana.org/assignments/service-names-port-numbers/ |

## Security

| Topic | Documentation |
|-------|---------------|
| OpenSSH | https://www.openssh.com/manual.html |
| `sshd_config` | `man sshd_config` |
| sudo / sudoers | https://www.sudo.ws/docs/ ; `man sudoers` |
| ufw | https://help.ubuntu.com/community/UFW |
| firewalld | https://firewalld.org/documentation/ |
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks |

## Package Management

| Tool | Documentation |
|------|---------------|
| APT / Debian packaging | https://www.debian.org/doc/manuals/debian-reference/ch02.en.html |
| DNF | https://dnf.readthedocs.io/ |

## Standards

| Standard | Link |
|----------|------|
| Filesystem Hierarchy Standard (FHS) | https://refspecs.linuxfoundation.org/fhs.shtml |
| POSIX (Open Group) | https://pubs.opengroup.org/onlinepubs/9699919799/ |

## DevOps / Cloud (Linux in context)

| Topic | Documentation |
|-------|---------------|
| AWS EC2 | https://docs.aws.amazon.com/ec2/ |
| Docker | https://docs.docker.com/ |
| Kubernetes | https://kubernetes.io/docs/ |
| cloud-init | https://cloudinit.readthedocs.io/ |

> All links point to official or long-established sources. If a link ever changes, search the tool name + "official documentation".

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [References](README.md) | ⬆️ Module: [References](README.md) | ➡️ Next: [Standards & RFCs](standards-and-rfcs.md) |
