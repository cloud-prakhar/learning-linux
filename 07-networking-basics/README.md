# Module 07 — Networking Basics

## What You Will Learn

- Core networking concepts (IP, DNS, ports, protocols).
- Checking IP, hostname, and DNS settings.
- Testing connectivity with ping, curl, wget.
- Understanding ports and sockets.
- Inspecting connections with ss, netstat, lsof.
- Troubleshooting common network problems.

## Why This Module Matters

Servers exist to talk over networks. When a site is down, an API times out, or a port is blocked, networking skills are what get you to the answer.

## Real-World Use Case

You'll check if a service is listening on a port, test whether a server is reachable, debug a DNS failure, and find which process owns a port — daily operations work.

## Topics Covered

| File | What It Covers |
|------|----------------|
| [networking-fundamentals.md](./networking-fundamentals.md) | IP, ports, protocols, OSI-lite |
| [ip-hostname-dns.md](./ip-hostname-dns.md) | ip, hostname, /etc/hosts, DNS |
| [ping-curl-wget.md](./ping-curl-wget.md) | Connectivity & HTTP testing |
| [ports-and-sockets.md](./ports-and-sockets.md) | What ports & sockets are |
| [netstat-ss-lsof.md](./netstat-ss-lsof.md) | Inspecting connections |
| [network-troubleshooting.md](./network-troubleshooting.md) | Fixing network issues |

## Learning Flow

```mermaid
flowchart LR
    A[Fundamentals] --> B[IP/Hostname/DNS]
    B --> C[ping/curl/wget]
    C --> D[Ports & sockets]
    D --> E[ss/netstat/lsof]
    E --> F[Troubleshooting]
```

## Hands-On Practice

Find your IP, resolve a domain, ping a host, curl a website, list listening ports, and identify which process owns one.

## Common Mistakes

- Assuming "ping works" means the app works (it only tests reachability).
- Confusing a DNS problem with a connectivity problem.

## Troubleshooting

- Can't reach a service → check DNS, then connectivity, then the port, then the app.
- Port in use → find the process with `ss`/`lsof`.

## Best Practices

- Debug layer by layer: DNS → IP reachability → port → application.
- Use `ss` (modern) over `netstat` (legacy).

## Quick Revision

- IP = address, port = door, DNS = phonebook.
- ping tests reachability; curl tests the actual service.
- `ss -ltnp` lists listening ports and their processes.

## Next Module

➡️ [08 — Storage & Disk Management](../08-storage-and-disk-management/).
