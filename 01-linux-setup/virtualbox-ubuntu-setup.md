# VirtualBox + Ubuntu Setup

## 1. What Is This?

**VirtualBox** is free software that runs a complete virtual computer on your machine. You install **Ubuntu** inside it to get a full Linux desktop, isolated from your real OS.

## 2. Why Is This Needed?

It gives you the **full Linux experience** — desktop, GUI apps, and terminal — in a safe sandbox you can snapshot, break, and reset without touching your main system.

## 3. Simple Layman Explanation

VirtualBox is like a **video game console emulator**, but instead of games it runs a whole operating system inside a window. Crash it? Just reset it. Your real computer is untouched.

## 4. Technical Explanation

VirtualBox is a **type-2 hypervisor**: it creates a virtual machine (virtual CPU, RAM, disk, network) on top of your existing OS. Ubuntu installs into that virtual disk and runs as if on real hardware.

## 5. Real-World Example

Trainers and students use VirtualBox to practice risky tasks (partitioning disks, breaking the bootloader) safely, because a broken VM can be deleted and rebuilt in minutes.

## 6. Diagram

```mermaid
flowchart LR
    Host[Your PC: Windows/macOS] --> VB[VirtualBox]
    VB --> VM[Virtual Machine]
    VM --> Ub[Ubuntu Desktop + Terminal]
```

## 7. Commands / Steps

This is a GUI setup. Steps:

```text
1. Download VirtualBox:  https://www.virtualbox.org/
2. Download Ubuntu ISO:  https://ubuntu.com/download/desktop
3. VirtualBox > New > name "ubuntu-lab", type Linux, version Ubuntu (64-bit)
4. Assign RAM: 2048-4096 MB ; CPUs: 2 ; Disk: 25 GB (dynamic)
5. Settings > Storage > attach the Ubuntu ISO to the virtual CD drive
6. Start the VM and follow the Ubuntu installer
7. Create your username and password
```

Inside the VM, verify:

```bash
lsb_release -a    # show Ubuntu version
free -h           # show memory assigned
nproc             # show CPU count
```

## 8. Command Explanation

- `lsb_release -a` → prints distribution name and version.
- `free -h` → shows memory in human-readable (`-h`) units.
- `nproc` → prints the number of processing units available.

## 9. Practice Tasks

1. Install VirtualBox and create the Ubuntu VM with the settings above.
2. After install, take a **snapshot** named "fresh-install".
3. Open the terminal and run `lsb_release -a`.

## 10. Common Mistakes

- Allocating too little RAM (≤1GB) → very slow. Give 2–4 GB.
- Forgetting to install **Guest Additions** for a smoother display.
- Not taking a snapshot before experiments.

## 11. Troubleshooting

- **VM won't start / VT-x error** → enable virtualization in BIOS.
- **Only 32-bit options shown** → enable virtualization; disable Hyper-V on Windows if it conflicts.
- **Tiny screen** → install Guest Additions and set scaled display.

## 12. Best Practices

- **Snapshot** before any risky change; revert if it breaks.
- Use a dynamically allocated disk to save space.
- Keep the VM updated with `sudo apt update && sudo apt upgrade`.

## 13. Quick Recap

- VirtualBox runs a full Ubuntu desktop in a safe window.
- Give it 2–4GB RAM and 25GB disk.
- Use snapshots to experiment fearlessly.

## 14. References

- VirtualBox: https://www.virtualbox.org/
- Ubuntu Desktop: https://ubuntu.com/download/desktop
- Ubuntu install tutorial: https://ubuntu.com/tutorials/install-ubuntu-desktop
