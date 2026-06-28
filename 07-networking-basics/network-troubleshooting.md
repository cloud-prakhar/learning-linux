# Network Troubleshooting

## 1. What Is This?

A layered method for diagnosing network problems: DNS failures, "connection refused", "connection timed out", and "port already in use".

## 2. Why Is This Needed?

Network issues are common and confusing. A consistent top-down method finds the real cause quickly instead of guessing.

## 3. Simple Layman Explanation

When a call won't connect, you check in order: do you have the right number (DNS)? Is the line reachable (ping)? Is anyone answering that extension (port)? Is the person actually working (app)?

## 4. Technical Explanation

Debug in layers, stopping at the first failure:
1. **DNS** — does the name resolve? (`getent hosts`, `dig`)
2. **Reachability** — is the host up? (`ping <ip>`)
3. **Port** — is the service listening/reachable? (`ss`, `curl`, `nc`)
4. **Application** — does the service actually respond correctly? (`curl -v`, logs)

---

## Scenario: DNS Not Resolving

### Problem
Names don't resolve, but IPs work.

### Symptoms
`curl: Could not resolve host`; `ping name` fails but `ping 8.8.8.8` works.

### Possible Causes
Bad/empty `/etc/resolv.conf`, DNS server down, wrong `/etc/hosts`.

### Commands to Check
```bash
ping -c 2 8.8.8.8
getent hosts google.com
cat /etc/resolv.conf
dig @8.8.8.8 google.com
```

### Step-by-Step Fix
1. Confirm IP reachability works (`ping 8.8.8.8`).
2. Check `/etc/resolv.conf` has a valid `nameserver`.
3. Test against a public resolver: `dig @8.8.8.8 name`.
4. Fix the resolver config or DNS server.

### Prevention
Use reliable DNS; don't hand-edit auto-managed resolv.conf.

---

## Scenario: Connection Refused

### Problem
You reach the host but the port rejects you instantly.

### Symptoms
`curl: (7) Connection refused`.

### Possible Causes
Nothing listening on that port, or the service bound to `127.0.0.1` only.

### Commands to Check
```bash
ss -ltnp | grep :<port>
sudo systemctl status <service>
curl -v http://127.0.0.1:<port>
```

### Step-by-Step Fix
1. Confirm the service is running (`systemctl status`).
2. Confirm it's listening on the expected address (`0.0.0.0` vs `127.0.0.1`).
3. Start the service / fix its bind address, then retry.

### Prevention
Ensure services listen on the right interface; monitor that they're up.

---

## Scenario: Connection Timed Out

### Problem
The connection hangs and eventually fails.

### Symptoms
`curl: (28) Connection timed out`; `ping` may also fail.

### Possible Causes
Firewall/security group blocking the port, wrong route, or host down.

### Commands to Check
```bash
ping -c 3 <host>
ss -ltnp | grep :<port>      # is it listening locally?
sudo ufw status              # local firewall (Module 12)
```

### Step-by-Step Fix
1. If `ping` fails → host/route/firewall issue at the network level.
2. If local `ss` shows it listening but remote times out → firewall/security group blocks the port.
3. Open the port in the firewall / cloud security group.

### Prevention
Document required ports; keep firewall rules in sync with services.

---

## Scenario: Port Already in Use

### Problem
A service won't start because its port is taken.

### Symptoms
`Address already in use` / `bind: address already in use`.

### Commands to Check
```bash
ss -ltnp | grep :<port>
sudo lsof -i :<port>
```

### Step-by-Step Fix
1. Identify the process holding the port.
2. Stop it (`systemctl stop` / `kill`) or reconfigure one service to a different port.
3. Start your service.

### Prevention
Avoid port clashes; document port assignments.

## 7. Commands (toolbox)

```bash
getent hosts NAME; dig NAME      # DNS
ping -c 3 HOST                   # reachability
ss -ltnp; sudo lsof -i :PORT     # ports
curl -v URL                      # application
nc -zv HOST PORT                 # test a remote port (netcat)
```

## 8. Command Explanation

- `nc -zv host port` → tests whether a remote TCP port is open without sending data.
- The rest are covered in their topic files; here they form a layered toolkit.

## 9. Practice Tasks

1. Simulate "connection refused": `curl http://127.0.0.1:9999` (nothing listening).
2. Start `python3 -m http.server 9999 &`, retry the curl (now works).
3. `nc -zv example.com 443` to test a remote port.
4. Break DNS test: `dig @8.8.8.8 thisdoesnotexist.example` and read the response.

## 10. Common Mistakes

- Skipping layers and guessing instead of testing DNS → reachability → port → app.
- Blaming the network when the app or firewall is the cause.
- Forgetting cloud **security groups** are a separate firewall from the host.

## 11. Troubleshooting

The scenarios above are the reference. Cross-check firewall rules in [Module 12](../12-linux-security-basics/firewall-basics-ufw-firewalld.md).

## 12. Best Practices

- Always debug top-down: DNS → ping → port → application.
- Use `curl -v` and `nc -zv` to pinpoint the failing layer.
- Keep firewall/security-group rules documented and minimal.

## 13. Quick Recap

- "Could not resolve" = DNS; "refused" = no listener/wrong bind; "timed out" = firewall/route; "in use" = port conflict.
- Test each layer in order.

## 14. References

- `man dig`, `man ss`, `man curl`, `man nc`
- [Module 12 firewall](../12-linux-security-basics/firewall-basics-ufw-firewalld.md)

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [netstat, ss, and lsof](netstat-ss-lsof.md) | ⬆️ Module: [Module 07 — Networking Basics](README.md) | ➡️ Next: [Module 08 — Storage & Disk Management](../08-storage-and-disk-management/README.md) |
