# Package Troubleshooting

## 1. What Is This?

How to fix the common package-manager errors: package not found, lock errors, broken dependencies, and repository/network failures.

## 2. Why Is This Needed?

Package errors block installs and updates, stalling everything else. A few known fixes resolve the vast majority.

## 3. Simple Layman Explanation

If the app store won't work, the usual causes are: outdated catalog, the store is already busy, a half-finished install, or the store's address is wrong. Each has a standard fix.

## 4. Technical Explanation

Most issues fall into four buckets:
1. **Stale index** → refresh it.
2. **Lock held** → another package process is running.
3. **Broken/half-configured packages** → repair them.
4. **Repo/network** → fix sources or connectivity.

---

## Scenario: "Unable to locate package" / "No match for argument"

### Problem
The package manager can't find a package you know exists.

### Symptoms
`E: Unable to locate package X` (apt) or `No match for argument: X` (dnf).

### Possible Causes
- Stale index (apt), typo, or the package needs an extra repository.

### Commands to Check
```bash
sudo apt update              # apt: refresh first
apt search <name>            # confirm exact name
dnf repolist                 # dnf: are repos enabled?
```

### Step-by-Step Fix
1. apt: run `sudo apt update`, retry.
2. Verify the exact package name via `apt search`/`dnf search`.
3. If it lives in an extra repo (e.g., EPEL/PPA), enable that repo, then retry.

### Prevention
Always `apt update` before installing; know which repos provide what.

---

## Scenario: "Could not get lock" (apt)

### Problem
apt refuses to run because a lock is held.

### Symptoms
`Could not get lock /var/lib/dpkg/lock-frontend ... resource temporarily unavailable`.

### Possible Causes
- Another apt process or automatic `unattended-upgrades` is running.

### Commands to Check
```bash
ps aux | grep -E "apt|dpkg|unattended"
```

### Step-by-Step Fix
1. Wait a minute — background updates often finish on their own.
2. Identify the process; let it complete.
3. Only if truly stuck and nothing is running: `sudo rm /var/lib/apt/lists/lock` and `sudo dpkg --configure -a` (last resort).

### Prevention
Don't run multiple apt commands at once; let auto-updates finish.

---

## Scenario: Broken Dependencies / Half-Configured

### Problem
A package failed to install/configure and now apt complains.

### Symptoms
`dpkg was interrupted`, `unmet dependencies`, or failed `apt` runs.

### Commands to Check
```bash
sudo apt --fix-broken install
sudo dpkg --configure -a
```

### Step-by-Step Fix
1. `sudo apt install -f` (fix broken).
2. `sudo dpkg --configure -a` (finish half-configured packages).
3. `sudo apt update && sudo apt upgrade`.

### Prevention
Don't interrupt installs; ensure enough disk space (Module 08).

---

## Scenario: Repo / Metadata Download Fails

### Problem
The manager can't reach repositories.

### Symptoms
`Failed to fetch`, `Could not resolve host`, `Failed to download metadata`.

### Possible Causes
- No internet, DNS failure, proxy, or a bad/expired repo.

### Commands to Check
```bash
ping -c 3 archive.ubuntu.com    # connectivity + DNS (Module 07)
cat /etc/apt/sources.list       # apt repos
ls /etc/yum.repos.d/            # dnf repos
```

### Step-by-Step Fix
1. Confirm network/DNS (Module 07).
2. Fix or disable the broken repo entry.
3. dnf cache issues: `sudo dnf clean all && sudo dnf makecache`.

### Prevention
Use reliable mirrors; keep repo configs valid; configure proxy if behind one.

## 9. Practice Tasks

1. Run `sudo apt update` and read the output sources.
2. Intentionally typo a package name and observe the "unable to locate" error, then fix the name.
3. Run `apt list --installed | wc -l` to count installed packages.

## 10. Common Mistakes

- Deleting lock files while apt is genuinely running (can corrupt state).
- Adding untrusted repos to "find" a package.
- Ignoring disk-full as a cause of failed installs.

## 11. Troubleshooting

This file *is* the troubleshooting reference. Cross-check network issues with [Module 07](../07-networking-basics/) and disk issues with [Module 08](../08-storage-and-disk-management/).

## 12. Best Practices

- Refresh index before installing; keep repos clean and trusted.
- Let automatic updates finish before manual runs.
- Maintain free disk space so installs don't fail mid-way.

## 13. Quick Recap

- Not found → `apt update` / check repo.
- Lock → wait for the other process.
- Broken → `apt install -f` + `dpkg --configure -a`.
- Repo fail → check network/DNS and sources.

## 14. References

- `man apt`, `man dpkg`, `man dnf`
- Ubuntu apt docs: https://ubuntu.com/server/docs/package-management
