# Lab 04 — Network Debugging

## Objective

Test DNS and connectivity, start a local service, confirm it's listening, identify the owning process, and reproduce a "connection refused".

## Scenario

A web service should be reachable but isn't. You'll work the network layers — DNS → reachability → port → application — to localize the problem.

## Prerequisites

- A Linux environment with `python3` (for a quick test server).
- Module 07 read.

## Steps & Commands

```bash
# 1. DNS + reachability
getent hosts github.com           # name -> IP
ping -c 3 8.8.8.8                  # raw reachability (bypasses DNS)
dig +short github.com              # DNS answer

# 2. Reproduce "connection refused" (nothing listening yet)
curl -sv http://127.0.0.1:8000 2>&1 | head -5    # expect: Connection refused

# 3. Start a local web server
python3 -m http.server 8000 &
SERVER_PID=$!
sleep 1

# 4. Confirm it's listening and who owns the port
ss -ltnp | grep 8000
sudo lsof -i :8000 2>/dev/null | head

# 5. Now connect successfully
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://127.0.0.1:8000

# 6. Test a remote port
nc -zv github.com 443 2>&1 || echo "nc not installed (optional)"

# 7. Stop the server
kill "$SERVER_PID"
curl -sv http://127.0.0.1:8000 2>&1 | head -3    # refused again
```

## Expected Output (samples)

```
$ curl -sv http://127.0.0.1:8000   (step 2)
*   Trying 127.0.0.1:8000...
* connect to 127.0.0.1 port 8000 failed: Connection refused

$ ss -ltnp | grep 8000
LISTEN 0 ... 0.0.0.0:8000 ... users:(("python3",pid=4321,fd=3))

$ curl ... (step 5)
HTTP 200
```

## Explanation

- `getent hosts` / `dig +short` → DNS resolution (Module 07). If these fail but `ping 8.8.8.8` works → DNS issue.
- Step 2's **Connection refused** → nothing is listening on 8000 yet (a port/application problem, not network).
- `python3 -m http.server 8000` → a one-line test web server.
- `ss -ltnp | grep 8000` → confirms the listener and its PID; `lsof -i :8000` confirms the owner.
- `curl -w "%{http_code}"` → verifies the **application** responds (HTTP 200).
- `nc -zv host 443` → tests a remote TCP port.

## Validation

- After step 3, `ss -ltnp` shows python3 listening on 8000 and `curl` returns `HTTP 200`.
- After step 7, `curl` returns "Connection refused" again.

## Cleanup

```bash
kill "$SERVER_PID" 2>/dev/null
pkill -f "http.server 8000" 2>/dev/null
```

## Troubleshooting

- **`ss` shows nothing for 8000** → the server didn't start; re-run step 3 and check for errors.
- **`Address already in use`** → another process holds 8000; `ss -ltnp | grep 8000`, then kill it or use a different port.
- **`nc: command not found`** → optional; install `netcat-openbsd` or skip step 6.
- **DNS fails** → check `/etc/resolv.conf` and try `dig @8.8.8.8 github.com` (Module 07).

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Lab 03 — Process & Service Debugging](lab-03-process-service-debugging.md) | ⬆️ Module: [Module 14 — Hands-On Labs](README.md) | ➡️ Next: [Lab 05 — Disk-Full Scenario](lab-05-disk-full-scenario.md) |
