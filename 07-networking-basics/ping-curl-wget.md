# ping, curl, wget

## 1. What Is This?

Three connectivity tools: **ping** (is the host reachable?), **curl** (talk to web services / APIs), and **wget** (download files).

## 2. Why Is This Needed?

These answer the everyday questions: "Is the server up?", "Is the API responding?", "Can I download this?" — the first checks in almost any network debugging.

## 3. Simple Layman Explanation

- `ping` = knock on the door and see if anyone answers.
- `curl` = actually have a conversation with the service inside.
- `wget` = fetch a package/file and bring it back.

## 4. Technical Explanation

| Tool | Layer | Use |
|------|-------|-----|
| `ping` | ICMP | Reachability + latency |
| `curl` | HTTP(S), many protocols | Test APIs/websites, headers, status codes |
| `wget` | HTTP(S)/FTP | Download files, mirror sites |

`ping` working ≠ app working. Always confirm the actual service with `curl`.

## 5. Real-World Example

A site is "down". `ping example.com` succeeds (server reachable), but `curl -I https://example.com` returns `503 Service Unavailable` — so the network is fine; the **application** is the problem. That distinction saves hours.

## 6. Diagram

```mermaid
flowchart LR
    Ping[ping: host reachable?] --> Curl[curl: service responding?]
    Curl --> Wget[wget: download works?]
```

## 7. Commands

```bash
ping -c 4 google.com             # 4 packets then stop
ping -c 4 8.8.8.8                # ping an IP (bypass DNS)
curl https://example.com         # fetch page body
curl -I https://example.com      # headers only (status code)
curl -v https://example.com      # verbose: connection details
curl -o page.html https://example.com   # save output to a file
curl -s -o /dev/null -w "%{http_code}\n" https://example.com  # just status code
wget https://example.com/file.tar.gz     # download a file
wget -O out.tar.gz <url>         # download to a specific name
```

## 8. Command Explanation

- `ping -c 4` → sends 4 packets (`-c`) instead of pinging forever; shows latency and packet loss.
- `curl -I` → HEAD request: returns status (200/301/404/503) and headers.
- `curl -v` → shows DNS, TCP, TLS, and request/response details — great for debugging.
- `curl -w "%{http_code}"` → prints only the HTTP status, ideal for scripts/health checks.
- `wget -O` → saves the download under a chosen filename.

## 9. Practice Tasks

1. `ping -c 4 8.8.8.8` and note latency/packet loss.
2. `curl -I https://example.com` — read the status line.
3. `curl -s -o /dev/null -w "%{http_code}\n" https://example.com`.
4. `wget https://raw.githubusercontent.com/.../README.md` (any public file).

## 10. Common Mistakes

- Treating a successful `ping` as proof the app works.
- Forgetting `-c` so `ping` runs endlessly (use `Ctrl+C`).
- Using `curl` without `-I`/`-v` and missing the status code.

## 11. Troubleshooting

- **ping by IP works, by name fails** → DNS issue (see ip-hostname-dns).
- **ping fails entirely** → host down, firewall blocking ICMP, or no route.
- **curl: "Could not resolve host"** → DNS. **"Connection refused"** → nothing listening on that port. **"timed out"** → firewall/routing.

## 12. Best Practices

- Use `curl` to verify the actual service, not just `ping`.
- In health checks, capture the HTTP status code with `curl -w`.
- Prefer HTTPS; verify certificates (don't blindly use `curl -k`).

## 13. Quick Recap

- `ping` = reachable? `curl` = service responding? `wget` = download.
- "Could not resolve" = DNS; "Connection refused" = port; "timed out" = firewall/route.

## 14. References

- curl: https://curl.se/docs/
- `man ping`, `man curl`, `man wget`

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [IP, Hostname, and DNS](ip-hostname-dns.md) | ⬆️ Module: [Module 07 — Networking Basics](README.md) | ➡️ Next: [Ports and Sockets](ports-and-sockets.md) |
