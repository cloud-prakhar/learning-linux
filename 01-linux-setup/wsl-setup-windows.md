# WSL Setup on Windows

## 1. What Is This?

**WSL (Windows Subsystem for Linux)** lets you run a real Linux distribution **inside Windows**, with a Linux terminal, without dual-booting or a heavy virtual machine.

## 2. Why Is This Needed?

It's the **fastest, safest** way for a Windows user to get a genuine Linux command line. You keep Windows for everything else and get Linux in a window.

## 3. Simple Layman Explanation

WSL is like having a **Linux room inside your Windows house**. You walk through a door (open the terminal) into a fully working Linux space, while the rest of your house stays Windows.

## 4. Technical Explanation

WSL 2 runs a real Linux kernel in a lightweight, optimized virtual machine that's tightly integrated with Windows. You get full system-call compatibility, your Linux files live in a Linux filesystem, and you can access Windows files too.

## 5. Real-World Example

Many developers and DevOps engineers on Windows laptops do all their Linux work — scripts, Docker, Git, SSH to cloud servers — entirely inside WSL.

## 6. Diagram

```mermaid
flowchart LR
    Win[Windows 10/11] --> WSL[WSL 2 layer]
    WSL --> Ubuntu[Ubuntu in a terminal]
    Ubuntu --> You[You run Linux commands]
```

## 7. Commands

Run these in **PowerShell as Administrator** first:

```powershell
wsl --install                 # installs WSL + Ubuntu (default)
wsl --list --online           # see available distros
wsl --install -d Ubuntu-22.04 # install a specific distro
wsl --status                  # check WSL version/status
```

Then inside the Ubuntu terminal:

```bash
sudo apt update && sudo apt upgrade -y   # update package lists & packages
whoami                                    # confirm your Linux user
```

## 8. Command Explanation

- `wsl --install` → enables WSL and installs Ubuntu in one step (Windows 10 2004+ / Windows 11).
- `wsl --list --online` → lists distros you can install.
- `-d Ubuntu-22.04` → `-d` selects a specific distribution.
- `sudo apt update` → refreshes the list of available packages (covered in Module 06).
- `&& apt upgrade -y` → upgrades installed packages; `-y` auto-confirms.

Expected output (first launch): it asks you to create a **UNIX username and password**. This is your Linux account, separate from Windows.

## 9. Practice Tasks

1. Install WSL with `wsl --install` and reboot if prompted.
2. Create your Linux username and password.
3. Run `sudo apt update` and confirm it completes without errors.
4. Run `pwd` and `whoami`.

## 10. Common Mistakes

- Forgetting to run PowerShell **as Administrator**.
- Confusing your Windows password with your new Linux password — they're separate.
- Editing Linux files from Windows tools in the wrong path (use `\\wsl$` paths if needed).

## 11. Troubleshooting

- **"WSL not recognized"** → update Windows; ensure you're on Windows 10 2004+ or 11.
- **Virtualization error** → enable virtualization (VT-x/SVM) in BIOS.
- **`wsl --install` fails** → run `wsl --update`, then retry.
- **Forgot Linux password** → reset via `wsl -u root` then `passwd <username>`.

## 12. Best Practices

- Keep WSL updated: `wsl --update`.
- Store your project files inside the Linux filesystem (`~/`) for best performance.
- Use **Windows Terminal** for a nicer tabbed experience.

## 13. Quick Recap

- `wsl --install` gives you Ubuntu inside Windows in minutes.
- You create a separate Linux username/password.
- It's the recommended starting point for Windows learners.

## 14. References

- WSL docs: https://learn.microsoft.com/windows/wsl/
- WSL install guide: https://learn.microsoft.com/windows/wsl/install

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Install Linux — Your Options](install-linux-options.md) | ⬆️ Module: [Module 01 — Linux Setup](README.md) | ➡️ Next: [VirtualBox + Ubuntu Setup](virtualbox-ubuntu-setup.md) |
