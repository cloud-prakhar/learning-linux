# mount and umount

## 1. What Is This?

**Mounting** attaches a filesystem (a disk, partition, USB, or network share) to a directory so you can access its files. **Unmounting** (`umount`) safely detaches it.

## 2. Why Is This Needed?

New disks, backup drives, and network shares are useless until mounted. Knowing how to mount (temporarily and at boot via `/etc/fstab`) is core storage admin.

## 3. Simple Layman Explanation

A disk is a book; a directory is a slot on your shelf. **Mounting** slides the book into a slot so you can read it. **Unmounting** removes it cleanly so you don't tear pages (corrupt data).

## 4. Technical Explanation

- `mount <device> <dir>` attaches a filesystem at a **mount point** (an existing directory).
- Mounts done with `mount` are **temporary** (lost on reboot).
- To make them **permanent**, add an entry to `/etc/fstab`.
- `umount` detaches; it fails if files are in use (a process has them open).

## 5. Real-World Example

You attach a new 100G volume to a cloud VM. Steps: format it (`mkfs.ext4 /dev/sdb`), create `/data`, `mount /dev/sdb /data`, then add it to `/etc/fstab` so it survives reboots. Now apps can store data on `/data`.

## 6. Diagram

```mermaid
flowchart LR
    Dev[/dev/sdb filesystem/] -->|mount| Dir[/data directory/]
    Dir --> Use[Read/write files]
    Use -->|umount| Detach[Safely detached]
```

## 7. Commands

```bash
lsblk -f                          # see devices & filesystems
sudo mkdir -p /data               # create a mount point
sudo mount /dev/sdb /data         # temporary mount
sudo mount -t ext4 /dev/sdb /data # specify filesystem type
mount | grep /data                # confirm it's mounted
df -h /data                       # check space on it
sudo umount /data                 # unmount
sudo umount -l /data              # lazy unmount (if busy)
blkid /dev/sdb                    # get the UUID for fstab
```

Permanent mount — add to `/etc/fstab` (use UUID):

```
UUID=xxxx-xxxx  /data  ext4  defaults  0  2
```

Then test without rebooting:

```bash
sudo mount -a        # mount everything in fstab; errors here mean a bad entry
```

## 8. Command Explanation

- `mount <dev> <dir>` → attaches now; not persistent.
- `mount -t ext4` → explicitly set filesystem type if auto-detect fails.
- `umount /data` → detaches; **must not be your current directory**.
- `umount -l` → lazy unmount; detaches when no longer busy (use carefully).
- `mount -a` → mounts all `/etc/fstab` entries — the safe way to test fstab before rebooting.
- `blkid` → gives the UUID to use in fstab (stable across reboots).

## 9. Practice Tasks

1. `lsblk -f` to inspect your devices.
2. (Safe demo) `mkdir /tmp/mp && sudo mount --bind /etc /tmp/mp`, then `ls /tmp/mp`, then `sudo umount /tmp/mp`.
3. `cat /etc/fstab` and map each line to `lsblk`.
4. Read `man fstab` for the field order.

## 10. Common Mistakes

- Editing `/etc/fstab` incorrectly → **system won't boot**. Always `mount -a` to test first.
- Using device names (`/dev/sdb`) in fstab instead of UUIDs (names can change).
- Trying to `umount` while you're `cd`'d inside the mount (it's "busy").

## 11. Troubleshooting

- **`umount: target is busy`** → find who's using it: `lsof +D /data` or `fuser -vm /data`; `cd` out; then unmount.
- **`mount: wrong fs type`** → specify `-t` or format the device first.
- **Bad fstab line broke boot** → boot to recovery, comment out the line, fix the UUID.

## 12. Best Practices

- Always test fstab changes with `sudo mount -a` before rebooting.
- Use UUIDs in fstab.
- Unmount cleanly to avoid corruption; never yank a busy mount.

## 13. Quick Recap

- `mount <dev> <dir>` (temporary), `/etc/fstab` (permanent), `umount` to detach.
- Test fstab with `mount -a`; use UUIDs.
- "Target is busy" → something has files open.

## 14. References

- `man mount`, `man umount`, `man fstab`, `man blkid`
