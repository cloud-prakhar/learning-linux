# Networking Cheatsheet

## IP / Interfaces

| Command | Purpose | Example |
|---------|---------|---------|
| `ip a` | Show IP addresses | `ip a` |
| `ip route` | Routing / default gateway | `ip route` |
| `hostname -I` | Quick IP list | `hostname -I` |
| `hostnamectl` | Host/OS info | `hostnamectl` |

## DNS

| Command | Purpose | Example |
|---------|---------|---------|
| `getent hosts name` | Resolve via system | `getent hosts github.com` |
| `dig name` | Detailed DNS query | `dig example.com` |
| `dig +short name` | Just the IP | `dig +short example.com` |
| `dig @8.8.8.8 name` | Query a specific resolver | `dig @8.8.8.8 example.com` |
| `cat /etc/resolv.conf` | Configured DNS servers | — |
| `cat /etc/hosts` | Local name overrides | — |

## Connectivity

| Command | Purpose | Example |
|---------|---------|---------|
| `ping -c4 host` | Reachability + latency | `ping -c4 8.8.8.8` |
| `curl -I url` | HTTP headers / status | `curl -I https://example.com` |
| `curl -v url` | Verbose connection | `curl -v https://example.com` |
| `curl -w '%{http_code}'` | Just status code | `curl -s -o /dev/null -w '%{http_code}\n' URL` |
| `wget url` | Download a file | `wget https://x/file.tgz` |
| `nc -zv host port` | Test a remote TCP port | `nc -zv example.com 443` |

## Ports / Sockets

| Command | Purpose | Example |
|---------|---------|---------|
| `ss -ltnp` | Listening TCP + process | `ss -ltnp` |
| `ss -tanp` | All TCP + process | `ss -tanp` |
| `ss -s` | Socket summary | `ss -s` |
| `sudo lsof -i :PORT` | Who owns a port | `sudo lsof -i :80` |
| `cat /etc/services` | Name↔port mapping | — |

## Common Ports

| Port | Service | Port | Service |
|------|---------|------|---------|
| 22 | SSH | 443 | HTTPS |
| 80 | HTTP | 53 | DNS |
| 3306 | MySQL | 5432 | PostgreSQL |
| 6379 | Redis | 25 | SMTP |

## Error → Meaning

| Error | Likely Cause |
|-------|--------------|
| Could not resolve host | DNS problem |
| Connection refused | Nothing listening / wrong port |
| Connection timed out | Firewall / route / host down |
| Address already in use | Port conflict |

## Debug Order

```
DNS (getent/dig) -> Reachability (ping) -> Port (ss/curl) -> App (curl -v/logs)
```

> Source: [Module 07](../07-networking-basics/).
