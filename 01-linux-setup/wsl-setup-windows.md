# WSL Setup on Windows

## 1. What Is This?

**WSL (Windows Subsystem for Linux)** lets you run a real Linux distribution **inside Windows**, with a Linux terminal, without dual-booting or a heavy virtual machine.

## 2. Why Is This Needed?

It's the **fastest, safest** way for a Windows user to get a genuine Linux command line. You keep Windows for everything else and get Linux in a window.

## 3. Simple Layman Explanation

WSL is like having a **Linux room inside your Windows house**. You walk through a door (open the terminal) into a fully working Linux space, while the rest of your house stays Windows.

## 4. Technical Explanation

WSL 2 runs a real Linux kernel in a lightweight, optimized virtual machine that's tightly integrated with Windows. You get full system-call compatibility, your Linux files live in a Linux filesystem, and you can access Windows files too.

## 5. How It Works Under the Hood

WSL 2 is a clever middle ground between "just an app" and "a full VM":

- Microsoft ships and maintains a **genuine Linux kernel** that runs inside a very thin, fast-booting virtual machine on Windows' **Hyper-V** platform. Because it's a real kernel, real Linux syscalls work — which is why Docker, `apt`, and compiled binaries behave exactly as on a native server (unlike the old WSL 1, which *translated* syscalls and had gaps).
- **Two filesystems, one big performance rule.** Your Linux files live in a Linux-native filesystem inside WSL (fast). Windows files are mounted under `/mnt/c`. Crossing the boundary is slow because every access is proxied between the two OSes — so keep project files in `~/` (Linux side), not `/mnt/c/...`. This is the single most common WSL performance mistake.
- **Separate identities.** WSL has its own Linux users and passwords, unrelated to your Windows login — because it's a separate OS with its own `/etc/passwd`. That's why first launch asks you to *create* a UNIX user.

So WSL feels like "Linux in a window" but is really a real kernel in a tuned VM, sharing files and networking with Windows through carefully managed bridges.

## 6. Diagram

```mermaid
flowchart LR
    Win[Windows 10/11] --> WSL[WSL 2 layer]
    WSL --> Ubuntu[Ubuntu in a terminal]
    Ubuntu --> You[You run Linux commands]
```

## 7. Real-World Examples

**1. The everyday case.** Many developers and DevOps engineers on Windows laptops do all their Linux work — scripts, Docker, Git, SSH to cloud servers — entirely inside WSL.

**2. First launch, on screen:**

```
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
Enter new UNIX username: prakhar
New password:
Retype new password:
passwd: password updated successfully
Installation successful!
prakhar@DESKTOP-8QF2:~$
```

That final prompt (`prakhar@DESKTOP...:~$`) means you're now inside real Linux.

**3. War story — the "why is my build so slow?" boundary trap.** An engineer cloned a Node project into `C:\Users\me\project` and ran `npm install` from WSL via `/mnt/c/Users/me/project`. It took 8 minutes. Moving the same project to `~/project` (the Linux filesystem) dropped it to under 1 minute. The cause was exactly Section 5: every file touch on `/mnt/c` was proxied across the Windows↔Linux boundary. Keeping code on the Linux side fixed it instantly.

## 8. Worked Walkthrough

From a fresh Windows machine to a working, updated Linux:

```
PS C:\> wsl --install
Installing: Windows Subsystem for Linux
Installing: Ubuntu
Downloading: Ubuntu
# (reboot if prompted)
```

After reboot, Ubuntu launches and asks for your UNIX user (see Section 7). Then, inside Ubuntu:

```
$ whoami
prakhar                                  # your new Linux user, not your Windows one
$ pwd
/home/prakhar                            # the fast, Linux-native home directory
$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Reading package lists... Done
$ wsl.exe --status        # (works from inside too) confirm you're on version 2
Default Version: 2
```

Notice `pwd` is `/home/prakhar`, **not** `/mnt/c/...` — that's where your projects should live (Section 5).

## 9. Commands

Run these in **PowerShell as Administrator** first:

```powershell
wsl --install                 # installs WSL + Ubuntu (default)
wsl --list --online           # see available distros
wsl --install -d Ubuntu-22.04 # install a specific distro
wsl --status                  # check WSL version/status
wsl --update                  # update the WSL kernel
```

Then inside the Ubuntu terminal:

```bash
sudo apt update && sudo apt upgrade -y   # update package lists & packages
whoami                                    # confirm your Linux user
```

Sample output for each (dummy values, for reference):

```text
PS C:\> wsl --status
Default Distribution: Ubuntu
Default Version: 2
Kernel version: 5.15.153.1-2

PS C:\> wsl --list --online
NAME               FRIENDLY NAME
Ubuntu             Ubuntu
Debian             Debian GNU/Linux
kali-linux         Kali Linux Rolling

$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
Reading package lists... Done

$ whoami
prakhar
```

## 10. Command Explanation

- `wsl --install` → enables WSL and installs Ubuntu in one step (Windows 10 2004+ / Windows 11).
- `wsl --list --online` → lists distros you can install.
- `-d Ubuntu-22.04` → `-d` selects a specific distribution.
- `wsl --status` / `wsl --update` → show the WSL version and update its kernel.
- `sudo apt update` → refreshes the list of available packages (covered in Module 06).
- `&& apt upgrade -y` → upgrades installed packages; `-y` auto-confirms.

First launch asks you to create a **UNIX username and password** — your Linux account, separate from Windows (Section 5).

## 11. In Production (DevOps Context)

- WSL is a mainstream **dev environment**: build and test on WSL, then deploy to identical Linux servers with confidence, because the kernel and tooling match.
- **Docker Desktop** on Windows runs its engine inside WSL 2 — so container work on Windows *is* Linux under the hood.
- SSH from WSL to cloud servers means one consistent toolchain (`ssh`, `scp`, keys) instead of Windows-specific tools.

## 12. Practice Tasks

1. Install WSL with `wsl --install` and reboot if prompted.
2. Create your Linux username and password.
3. Run `sudo apt update` and confirm it completes without errors.
4. Run `pwd` and confirm you're in `/home/<you>`, not `/mnt/c/...`.

## 13. Common Mistakes

- Forgetting to run PowerShell **as Administrator**.
- Confusing your Windows password with your new Linux password — they're separate.
- Storing projects under `/mnt/c/...` and suffering slow file access (the war story) — keep them in `~/`.

## 14. Troubleshooting

- **"WSL not recognized"** → update Windows; ensure you're on Windows 10 2004+ or 11.
- **Virtualization error** → enable virtualization (VT-x/SVM) in BIOS.
- **`wsl --install` fails** → run `wsl --update`, then retry.
- **Forgot Linux password** → reset via `wsl -u root` then `passwd <username>`.

## 15. Best Practices

- Keep WSL updated: `wsl --update`.
- Store your project files inside the Linux filesystem (`~/`) for best performance.
- Use **Windows Terminal** for a nicer tabbed experience.

## 16. Connects To

- **Prev:** [Install Linux — Your Options](install-linux-options.md). **Next:** [VirtualBox + Ubuntu Setup](virtualbox-ubuntu-setup.md).
- **Start using it:** [Terminal Basics](terminal-basics.md).
- **Packages you'll run:** [Module 06 — Package Management](../06-package-management/README.md).
- **Docker on WSL:** [Linux for Docker](../13-real-world-linux-for-devops/linux-for-docker.md).

## 17. Quick Recap

- `wsl --install` gives you Ubuntu inside Windows in minutes.
- WSL 2 = a real Linux kernel in a tiny VM; keep files in `~/`, not `/mnt/c`, for speed.
- You create a separate Linux username/password.
- It's the recommended starting point for Windows learners.

## 18. References

- WSL docs: https://learn.microsoft.com/windows/wsl/
- WSL install guide: https://learn.microsoft.com/windows/wsl/install

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Install Linux — Your Options](install-linux-options.md) | ⬆️ Module: [Module 01 — Linux Setup](README.md) | ➡️ Next: [VirtualBox + Ubuntu Setup](virtualbox-ubuntu-setup.md) |
