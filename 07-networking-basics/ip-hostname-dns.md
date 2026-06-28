# IP, Hostname, and DNS

## 1. What Is This?

Managing a host's **IP address**, its **hostname** (its name), and **DNS** (the system that turns names into IPs).

## 2. Why Is This Needed?

You constantly need to know a server's IP, set or read its hostname, and verify that name resolution works — DNS is behind a huge fraction of "it won't connect" issues.

## 3. Simple Layman Explanation

- **IP** = your house's coordinates.
- **Hostname** = your house's nickname ("the blue house").
- **DNS** = the directory that maps nicknames and names to coordinates.

## 4. Technical Explanation

- `ip` is the modern tool for addresses/routes (replaces old `ifconfig`).
- Hostname is stored in `/etc/hostname`; `hostnamectl` manages it on systemd systems.
- Name resolution order: `/etc/hosts` (local overrides) → DNS servers in `/etc/resolv.conf`.
- Tools: `dig`, `nslookup`, `host`, `getent` query DNS.

## 5. Real-World Example

An app can't reach `db.internal`. `getent hosts db.internal` returns nothing → DNS isn't resolving it. You add a temporary `/etc/hosts` entry to confirm, then fix the real DNS record.

## 6. Diagram

```mermaid
flowchart LR
    App[App asks for db.internal] --> Hosts{/etc/hosts?}
    Hosts -->|found| IP[Use that IP]
    Hosts -->|not found| DNS[(DNS server)]
    DNS --> IP
```

## 7. Commands

```bash
ip a                         # show IP addresses
ip route                     # routing table / default gateway
hostname                     # current hostname
hostnamectl                  # detailed host info
sudo hostnamectl set-hostname web01   # set hostname
cat /etc/hosts               # local name->IP overrides
cat /etc/resolv.conf         # configured DNS servers
getent hosts example.com     # resolve via system resolver
dig example.com              # detailed DNS query
nslookup example.com         # simple DNS query
```

## 8. Command Explanation

- `ip a` / `ip route` → addresses and the default gateway (how packets leave).
- `hostnamectl set-hostname web01` → permanently sets the hostname.
- `/etc/hosts` → manual name→IP mappings, checked **before** DNS.
- `/etc/resolv.conf` → lists DNS servers (`nameserver 8.8.8.8`).
- `dig example.com` → shows the resolved IP, the answer section, and timing.
- `getent hosts` → uses the full system resolution path (hosts + DNS).

`dig` answer (trimmed):

```
;; ANSWER SECTION:
example.com.   3600  IN  A  93.184.216.34
```

## 9. Practice Tasks

1. `ip a` and `ip route` — find your IP and gateway.
2. `hostname` and `hostnamectl`.
3. `cat /etc/resolv.conf` — note your DNS servers.
4. `dig github.com` and `getent hosts github.com`.
5. Add a test line to `/etc/hosts` (e.g., `127.0.0.1 myapp.local`) and `ping myapp.local`.

## 10. Common Mistakes

- Editing `/etc/resolv.conf` directly when it's auto-managed (changes get overwritten).
- Forgetting `/etc/hosts` is checked first — a stale entry there can mask real DNS.
- Using deprecated `ifconfig` instead of `ip`.

## 11. Troubleshooting

- **Name fails, IP works** → DNS issue: check `/etc/resolv.conf`, try `dig @8.8.8.8 name`.
- **Wrong IP returned** → stale `/etc/hosts` entry or cached DNS.
- **No default route** → `ip route` shows no `default`; networking misconfigured.

## 12. Best Practices

- Use `/etc/hosts` only for deliberate local overrides; document them.
- Test DNS against a known public resolver (`dig @8.8.8.8`) to isolate issues.
- Set meaningful hostnames (e.g., `web01-prod`).

## 13. Quick Recap

- `ip a`/`ip route` for addresses; `hostnamectl` for the name.
- Resolution: `/etc/hosts` → DNS (`/etc/resolv.conf`).
- `dig`/`getent` to test name resolution.

## 14. References

- `man ip`, `man hostnamectl`, `man dig`, `man resolv.conf`
- systemd-resolved: https://www.freedesktop.org/software/systemd/man/systemd-resolved.html
