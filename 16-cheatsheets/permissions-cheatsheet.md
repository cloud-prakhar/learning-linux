# Permissions Cheatsheet

## Reading `ls -l`

```
-rwxr-xr--  1 alice devs 220 Jun 28 file
│└┬┘└┬┘└┬┘
│ │  │  └ others: r--
│ │  └─── group:  r-x
│ └────── owner:  rwx
└──────── type:  - file, d dir, l link
```

## rwx ↔ Octal

| Symbolic | Octal | Meaning |
|----------|-------|---------|
| `---` | 0 | none |
| `r--` | 4 | read |
| `rw-` | 6 | read+write |
| `r-x` | 5 | read+execute |
| `rwx` | 7 | full |

Add per class: `rwxr-xr--` = **754**.

## Common Permission Sets

| Octal | Use |
|-------|-----|
| `600` | Private files, SSH keys (`rw-------`) |
| `644` | Normal files / web files (`rw-r--r--`) |
| `640` | Group-readable config |
| `755` | Directories, executables (`rwxr-xr-x`) |
| `750` | Group-only executables/dirs |
| `700` | Private directory |

## chmod

| Command | Purpose | Example |
|---------|---------|---------|
| `chmod 644 f` | Set numeric perms | `chmod 644 index.html` |
| `chmod +x f` | Add execute | `chmod +x script.sh` |
| `chmod u+x f` | Execute for owner | `chmod u+x run.sh` |
| `chmod go-w f` | Remove write (group+others) | `chmod go-w secret` |
| `chmod -R 755 d` | Recursive | `chmod -R 755 /var/www` |

Set files vs dirs separately:
```bash
find dir -type f -exec chmod 644 {} \;
find dir -type d -exec chmod 755 {} \;
```

## chown / chgrp

| Command | Purpose | Example |
|---------|---------|---------|
| `chown user f` | Change owner | `sudo chown alice f` |
| `chown user:grp f` | Owner + group | `sudo chown www-data:www-data f` |
| `chown -R u:g d` | Recursive | `sudo chown -R deploy:deploy /app` |
| `chgrp grp f` | Change group | `sudo chgrp devs f` |

## Special / Inspect

| Command | Purpose |
|---------|---------|
| `umask` | Default-perms mask (022 → 644/755) |
| `chmod 2770 d` | setgid dir (new files inherit group) |
| `stat f` | Detailed perms (symbolic + octal) |
| `namei -l /path` | Perms at every path level |
| `id` | Your UID/GID/groups |

## Quick Fixes

| Problem | Fix |
|---------|-----|
| Script won't run | `chmod +x script.sh` |
| SSH key too open | `chmod 600 ~/.ssh/id_*` |
| Wrong owner | `sudo chown user:group file` |
| Can list dir but not `cd` | dir needs `x`: `chmod +x dir` |

> ⚠️ Avoid `chmod 777` — it lets everyone modify the file.
> Source: [Module 04](../04-users-groups-permissions/).
