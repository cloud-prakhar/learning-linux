# Standards & RFCs

The official specifications behind the tools and protocols in this repo. These are the **primary sources** — the documents that *define* how Linux and the internet actually work. You don't need to read them cover to cover; use them to settle "how does this really work?" questions.

## What Is an RFC?

An **RFC (Request for Comments)** is an official, numbered document published by the **IETF (Internet Engineering Task Force)** that defines internet protocols and standards. They are the authoritative source for TCP, DNS, HTTP, SSH, and more. Read them at the official publisher: **https://www.rfc-editor.org/**.

> Tip: an RFC may be **obsoleted** by a newer one. The rfc-editor page for each RFC shows its status and what replaced it. This page cites the current/standard versions.

---

## Linux & OS Standards

| Standard | What It Defines | Official Source |
|----------|-----------------|-----------------|
| Filesystem Hierarchy Standard (FHS) | The `/etc`, `/var`, `/home`… directory layout | https://refspecs.linuxfoundation.org/fhs.shtml |
| POSIX (IEEE Std 1003.1) | Portable OS interface: shell & utilities, system APIs | https://pubs.opengroup.org/onlinepubs/9699919799/ |
| Linux Standard Base (LSB) | Cross-distro binary compatibility | https://refspecs.linuxfoundation.org/lsb.shtml |
| GNU Coding Standards | Conventions for GNU tools | https://www.gnu.org/prep/standards/ |
| Single UNIX Specification | UNIX behavior (aligns with POSIX) | https://unix.org/online.html |

Relevant modules: [02 Linux Basics](../02-linux-basics/), [03 Files & Directories](../03-files-and-directories/).

---

## Networking RFCs

| RFC | Protocol | Notes | Link |
|-----|----------|-------|------|
| RFC 791 | IPv4 | The Internet Protocol | https://www.rfc-editor.org/rfc/rfc791 |
| RFC 8200 | IPv6 | Current IPv6 spec | https://www.rfc-editor.org/rfc/rfc8200 |
| RFC 9293 | TCP | Current TCP spec (obsoletes RFC 793) | https://www.rfc-editor.org/rfc/rfc9293 |
| RFC 768 | UDP | User Datagram Protocol | https://www.rfc-editor.org/rfc/rfc768 |
| RFC 1034 | DNS — concepts | How DNS is structured | https://www.rfc-editor.org/rfc/rfc1034 |
| RFC 1035 | DNS — implementation | Message format & records | https://www.rfc-editor.org/rfc/rfc1035 |
| RFC 1918 | Private IPv4 ranges | 10/8, 172.16/12, 192.168/16 | https://www.rfc-editor.org/rfc/rfc1918 |
| RFC 2131 | DHCP | Dynamic address assignment | https://www.rfc-editor.org/rfc/rfc2131 |
| RFC 826 | ARP | IP → MAC resolution | https://www.rfc-editor.org/rfc/rfc826 |
| RFC 9110 | HTTP semantics | Methods, status codes (obsoletes 2616/723x) | https://www.rfc-editor.org/rfc/rfc9110 |
| RFC 5321 | SMTP | Email transport | https://www.rfc-editor.org/rfc/rfc5321 |

Relevant module: [07 Networking Basics](../07-networking-basics/).

---

## Security & SSH RFCs

| RFC | Topic | Link |
|-----|-------|------|
| RFC 4251 | SSH protocol architecture | https://www.rfc-editor.org/rfc/rfc4251 |
| RFC 4252 | SSH authentication protocol | https://www.rfc-editor.org/rfc/rfc4252 |
| RFC 4253 | SSH transport layer protocol | https://www.rfc-editor.org/rfc/rfc4253 |
| RFC 8446 | TLS 1.3 (HTTPS encryption) | https://www.rfc-editor.org/rfc/rfc8446 |
| RFC 6749 | OAuth 2.0 (auth, used by cloud/IAM) | https://www.rfc-editor.org/rfc/rfc6749 |

Relevant module: [12 Linux Security Basics](../12-linux-security-basics/).

---

## File Formats & Time

| Standard | Topic | Link |
|----------|-------|------|
| RFC 4180 | CSV format (used by scripts/reports) | https://www.rfc-editor.org/rfc/rfc4180 |
| RFC 3339 | Date/time format (ISO 8601 profile) | https://www.rfc-editor.org/rfc/rfc3339 |
| RFC 5322 | Internet message (email) format | https://www.rfc-editor.org/rfc/rfc5322 |

Relevant modules: [10 Shell Scripting](../10-shell-scripting/), [11 Automation & Cron](../11-automation-and-cron/).

---

## How to Use These

1. **Start with the repo + man pages** for practical understanding.
2. **Consult the RFC/standard** when you need the precise, authoritative behavior (e.g., "what exactly is a private IP range?" → RFC 1918).
3. RFCs are dense — read the **Introduction** and **section headings** first; dive deeper only where needed.
4. Always confirm an RFC's **status** on rfc-editor.org (it may be updated/obsoleted).

> These are the ground truth. When a blog post and an RFC disagree, the RFC wins.

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Official Documentation](official-documentation.md) | ⬆️ Module: [References](README.md) | ➡️ Next: [Official Reading List by Module](reading-list-by-module.md) |
