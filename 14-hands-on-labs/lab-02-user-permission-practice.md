# Lab 02 — User & Permission Practice

## Objective

Create a user and group, set ownership and permissions on a shared directory, and reproduce + fix a "Permission denied" error.

## Scenario

Two teammates need shared access to a project directory, but private files must stay protected. You'll model this with users, a group, and correct permissions.

## Prerequisites

- A Linux environment where you have `sudo`.
- Module 04 read.
- ⚠️ Use a test/throwaway machine (this lab creates users).

## Steps & Commands

```bash
# 1. Create a group and two users
sudo groupadd devteam
sudo useradd -m -s /bin/bash alice
sudo useradd -m -s /bin/bash bob
sudo usermod -aG devteam alice
sudo usermod -aG devteam bob
id alice ; id bob

# 2. Create a shared directory owned by the group
sudo mkdir -p /srv/shared
sudo chown root:devteam /srv/shared
sudo chmod 2770 /srv/shared        # rwx for owner+group, setgid (2)
ls -ld /srv/shared

# 3. Test: alice creates a file, group can access it
sudo -u alice bash -c 'echo "hello from alice" > /srv/shared/notes.txt'
ls -l /srv/shared
sudo -u bob bash -c 'cat /srv/shared/notes.txt'

# 4. Reproduce a permission error
sudo mkdir -p /srv/private
sudo chmod 700 /srv/private
sudo -u alice bash -c 'ls /srv/private'   # should FAIL: Permission denied

# 5. Fix it by granting the group access
sudo chown root:devteam /srv/private
sudo chmod 750 /srv/private
sudo -u alice bash -c 'ls /srv/private'   # now works (read+traverse)
```

## Expected Output (samples)

```
$ ls -ld /srv/shared
drwxrws--- 2 root devteam 4096 Jun 28 10:00 /srv/shared

$ sudo -u alice ls /srv/private
ls: cannot open directory '/srv/private': Permission denied   # step 4

$ sudo -u alice ls /srv/private
(no error - lists contents)                                   # step 5
```

## Explanation

- `usermod -aG devteam alice` → adds alice to the group (`-a` append, crucial).
- `chmod 2770` → owner+group get `rwx`; the leading `2` is **setgid**, so new files inherit the `devteam` group automatically.
- `sudo -u alice bash -c '...'` → runs a command **as alice** to test her access.
- Step 4 fails because `700` gives no group/other access and `/srv/private` is owned by root.
- `chmod 750` + group ownership → grants the group read + traverse (`x`), fixing it.

## Validation

- `notes.txt` in `/srv/shared` is group `devteam`; bob can read it.
- After step 5, alice can list `/srv/private`; before, she couldn't.

## Cleanup

```bash
sudo rm -rf /srv/shared /srv/private
sudo userdel -r alice
sudo userdel -r bob
sudo groupdel devteam
```

## Troubleshooting

- **Group membership "not working"** → `sudo -u` starts a fresh session so it applies; for a real login, log out/in or use `newgrp`.
- **`useradd: user already exists`** → it's already created; `id alice` to confirm, or pick another name.
- **Still denied after chmod** → check a parent directory also allows traverse (`namei -l /srv/private`).

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Lab 01 — Basic Commands & Navigation](lab-01-basic-commands.md) | ⬆️ Module: [Module 14 — Hands-On Labs](README.md) | ➡️ Next: [Lab 03 — Process & Service Debugging](lab-03-process-service-debugging.md) |
