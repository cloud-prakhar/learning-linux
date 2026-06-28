# SSH Basics

## 1. What Is This?

**SSH (Secure Shell)** is the encrypted protocol for logging into and running commands on remote Linux servers. **Key-based authentication** uses a key pair instead of a password.

## 2. Why Is This Needed?

SSH is how you reach almost every cloud/remote server. Doing it securely (keys, no root login) is the single highest-impact security step you can take.

## 3. Simple Layman Explanation

SSH is a **secure phone line** to your server — everything said is scrambled so eavesdroppers learn nothing. A key pair is like a **special lock and key**: the lock (public key) sits on the server; only your private key opens it. Far safer than a guessable password.

## 4. Technical Explanation

- A **key pair**: a **private key** (kept secret on your machine) and a **public key** (placed on the server in `~/.ssh/authorized_keys`).
- The server proves you hold the private key without it ever being sent.
- Server config lives in `/etc/ssh/sshd_config`; hardening disables password login and root login.
- Default port is 22.

## 5. Real-World Example

You generate a key pair, copy the public key to a new EC2 instance, and from then on `ssh ubuntu@server` logs in instantly and securely — no password, immune to brute-force.

## 6. Diagram

```mermaid
flowchart LR
    Priv[Your private key] -->|proves identity| SSH[ssh client]
    SSH -->|port 22, encrypted| Server[sshd]
    Server --> Auth[checks authorized_keys public key]
    Auth -->|match| Shell[Logged in]
```

## 7. Commands

```bash
ssh-keygen -t ed25519 -C "you@example.com"   # generate a modern key pair
ls ~/.ssh                                     # id_ed25519 (private), id_ed25519.pub (public)
ssh-copy-id user@server                       # install your public key on the server
ssh user@server                               # connect
ssh -i ~/.ssh/id_ed25519 user@server          # specify a key explicitly
ssh -p 2222 user@server                       # non-default port
chmod 600 ~/.ssh/id_ed25519                   # private key must be owner-only
sudo systemctl reload ssh                     # apply sshd config changes
```

Hardening in `/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
```

## 8. Command Explanation

- `ssh-keygen -t ed25519` → creates a strong, modern key pair; add a passphrase when prompted.
- `ssh-copy-id` → appends your public key to the server's `authorized_keys`.
- `ssh -i <key>` → choose which private key to use.
- `chmod 600` on the private key → SSH refuses keys that are too open.
- Editing `sshd_config` then `systemctl reload ssh` → disables root and password login (keep a working session open while testing!).

## 9. Practice Tasks

1. Generate a key pair with `ssh-keygen -t ed25519`.
2. Inspect `~/.ssh`; identify the public vs private key.
3. (On a test server) `ssh-copy-id` and log in without a password.
4. Confirm `chmod 600` on the private key.

## 10. Common Mistakes

- Sharing or committing the **private** key (never — only the `.pub` is shared).
- Disabling password auth before confirming key login works → lockout.
- Loose private key permissions → "UNPROTECTED PRIVATE KEY" error.

## 11. Troubleshooting

- **Permission denied (publickey)** → wrong key/user, or public key not in `authorized_keys`.
- **Connection refused** → sshd not running or firewall blocks the port (`systemctl status ssh`, firewall).
- **"UNPROTECTED PRIVATE KEY FILE"** → `chmod 600` the private key.
- **Connection timed out** → security group/firewall, or wrong IP.

## 12. Best Practices

- Use key-based auth; disable password and root login.
- Protect private keys with a passphrase; never share them.
- Restrict SSH to known IPs in the firewall/security group.
- Keep a second active session open when changing sshd config.

## 13. Quick Recap

- SSH = encrypted remote access; keys > passwords.
- Public key on server, private key stays with you (`chmod 600`).
- Harden: `PermitRootLogin no`, `PasswordAuthentication no`.

## 14. References

- OpenSSH: https://www.openssh.com/manual.html
- `man ssh`, `man ssh-keygen`, `man sshd_config`
