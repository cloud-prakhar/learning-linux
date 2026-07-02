# VirtualBox + Ubuntu Setup

## 1. What Is This?

**VirtualBox** is free software that runs a complete virtual computer on your machine. You install **Ubuntu** inside it to get a full Linux desktop, isolated from your real OS.

## 2. Why Is This Needed?

It gives you the **full Linux experience** — desktop, GUI apps, and terminal — in a safe sandbox you can snapshot, break, and reset without touching your main system.

## 3. Simple Layman Explanation

VirtualBox is like a **video game console emulator**, but instead of games it runs a whole operating system inside a window. Crash it? Just reset it. Your real computer is untouched.

## 4. Technical Explanation

VirtualBox is a **type-2 hypervisor**: it creates a virtual machine (virtual CPU, RAM, disk, network) on top of your existing OS. Ubuntu installs into that virtual disk and runs as if on real hardware.

## 5. How It Works Under the Hood

A virtual machine works by **lying convincingly to the guest OS**. VirtualBox presents Ubuntu with fake-but-standard hardware — a virtual CPU, a virtual disk that's really just a big `.vdi` file on your Windows/macOS drive, a virtual network card, a virtual display. Ubuntu installs and boots exactly as if that hardware were real.

The key mechanism is the **hypervisor**:

- Modern CPUs have virtualization extensions (**Intel VT-x / AMD-V**). VirtualBox uses them to run the guest's code almost directly on your real CPU at near-native speed, only stepping in ("trapping") when the guest tries to touch hardware — then it emulates the result. This is why enabling VT-x in BIOS is mandatory and why, without it, you either can't start the VM or only see 32-bit options.
- Being **type-2** means VirtualBox sits *on top of* your host OS (unlike type-1 hypervisors that run on bare metal). Convenient, but it's why a VM is a bit slower than the host and why the guest's RAM is *carved out* of your host's RAM — give a VM 4 GB and your host has 4 GB less until you shut it down.
- A **snapshot** freezes the entire virtual disk + memory state to disk. Restoring rewinds the whole machine to that instant — the superpower that lets you break things fearlessly.

That mental model explains all the setup advice: enough RAM (host and guest share it), enough disk (the `.vdi` grows), and snapshots before risky experiments.

## 6. Diagram

```mermaid
flowchart LR
    Host[Your PC: Windows/macOS] --> VB[VirtualBox]
    VB --> VM[Virtual Machine]
    VM --> Ub[Ubuntu Desktop + Terminal]
```

## 7. Real-World Examples

**1. The everyday case.** Trainers and students use VirtualBox to practice risky tasks (partitioning disks, breaking the bootloader) safely, because a broken VM can be deleted and rebuilt in minutes.

**2. Verifying the resources the hypervisor handed the guest:**

```
$ nproc
2                                  # 2 virtual CPUs, as configured in VM settings
$ free -h | head -2
               total        used        free
Mem:            3.8Gi       0.6Gi       3.0Gi     # ~4 GB carved from the host
$ lsb_release -d
Description:    Ubuntu 22.04.4 LTS
```

These numbers match what you set in VirtualBox — proof the guest sees exactly the fake hardware you assigned.

**3. War story — the snapshot that saved a class.** A trainer demoing disk partitioning ran a wrong command that corrupted the guest's bootloader; Ubuntu wouldn't boot. Instead of a 40-minute reinstall, they clicked **Restore Snapshot "fresh-install"** and were back to a working desktop in under a minute. The takeaway from Section 5 in practice: snapshot *before* anything risky, and a broken VM is never a real problem.

## 8. Worked Walkthrough

The setup is GUI-driven; here's the full path with the key numbers:

```text
1. Install VirtualBox      → https://www.virtualbox.org/
2. Download Ubuntu ISO     → https://ubuntu.com/download/desktop
3. VirtualBox > New        → Name: ubuntu-lab, Type: Linux, Version: Ubuntu (64-bit)
4. Memory                  → 2048–4096 MB   (remember: carved from host RAM)
5. CPUs                    → 2
6. Disk                    → 25 GB, dynamically allocated (.vdi grows as needed)
7. Settings > Storage      → attach the Ubuntu ISO to the virtual optical drive
8. Start                   → follow the Ubuntu installer, create username + password
9. IMMEDIATELY             → take a snapshot named "fresh-install"
```

First boot into the desktop, open a terminal, and confirm the machine is what you built:

```
$ lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
$ free -h
               total        used        free
Mem:            3.8Gi       0.6Gi       3.0Gi
$ nproc
2
```

If RAM/CPU match your settings, the hypervisor did its job — now go take that snapshot.

## 9. Commands / Steps

Inside the VM, verify the environment:

```bash
lsb_release -a    # show Ubuntu version
free -h           # show memory assigned to the guest
nproc             # show CPU count assigned to the guest
df -h /           # show virtual disk usage
```

Sample output for each (dummy values, for reference):

```text
$ lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
Release:        22.04
Codename:       jammy

$ free -h
               total        used        free      shared  buff/cache   available
Mem:            3.8Gi       0.6Gi       3.0Gi       40Mi        200Mi       3.0Gi

$ nproc
2

$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        24G  7.2G   16G  32% /
```

## 10. Command Explanation

- `lsb_release -a` → prints distribution name and version.
- `free -h` → shows memory in human-readable (`-h`) units — the slice carved from the host.
- `nproc` → prints the number of virtual processing units assigned.
- `df -h /` → shows how full the virtual disk (`.vdi`) is.

## 11. In Production (DevOps Context)

- The VM concept here *is* cloud computing in miniature — EC2/GCE instances are VMs on a provider's (type-1) hypervisor.
- **Snapshots** mirror cloud "AMI/image" and VM-restore workflows used to roll back bad changes.
- Practicing risky operations (partitioning, service configs) in a snapshotted VM builds the exact instincts you'll use before touching a production server.

## 12. Practice Tasks

1. Install VirtualBox and create the Ubuntu VM with the settings in Section 8.
2. After install, take a **snapshot** named "fresh-install".
3. Open the terminal and run `lsb_release -a`, `nproc`, and `free -h`; confirm they match your settings.
4. Deliberately create a junk file, then restore the snapshot and confirm it's gone.

## 13. Common Mistakes

- Allocating too little RAM (≤1GB) → very slow, because host and guest share memory. Give 2–4 GB.
- Forgetting to install **Guest Additions** for a smoother display and clipboard.
- Not taking a snapshot before experiments (the war story is why you always do).

## 14. Troubleshooting

- **VM won't start / VT-x error** → enable virtualization in BIOS (Section 5 explains why it's required).
- **Only 32-bit options shown** → enable virtualization; disable Hyper-V on Windows if it conflicts.
- **Tiny screen** → install Guest Additions and set scaled display.

## 15. Best Practices

- **Snapshot** before any risky change; revert if it breaks.
- Use a dynamically allocated disk to save space.
- Keep the VM updated with `sudo apt update && sudo apt upgrade`.

## 16. Connects To

- **Prev:** [WSL Setup on Windows](wsl-setup-windows.md). **Next:** [Cloud Linux Server (AWS EC2)](cloud-linux-server.md).
- **Compare options:** [Install Linux — Your Options](install-linux-options.md).
- **Then start using it:** [Terminal Basics](terminal-basics.md).
- **Practice disks safely:** [Module 08 — Storage & Disk](../08-storage-and-disk-management/README.md).

## 17. Quick Recap

- VirtualBox runs a full Ubuntu desktop in a safe window via a type-2 hypervisor + VT-x/AMD-V.
- The guest's RAM/CPU/disk are carved from (or backed by) the host — size them sensibly.
- Give it 2–4GB RAM and 25GB disk; use snapshots to experiment fearlessly.

## 18. References

- VirtualBox: https://www.virtualbox.org/
- Ubuntu Desktop: https://ubuntu.com/download/desktop
- Ubuntu install tutorial: https://ubuntu.com/tutorials/install-ubuntu-desktop

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [WSL Setup on Windows](wsl-setup-windows.md) | ⬆️ Module: [Module 01 — Linux Setup](README.md) | ➡️ Next: [Cloud Linux Server (AWS EC2)](cloud-linux-server.md) |
