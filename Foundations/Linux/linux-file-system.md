# Linux File System Hierarchy Explained

> A comprehensive guide to understanding the Linux Filesystem Hierarchy Standard (FHS) — from `/` to `/var` and everything in between.

---

## Introduction

> **"Everything in Linux is a file."**


Linux is an **open-source, Unix-like operating system** that powers everything from smartphones and laptops to servers, supercomputers, and embedded systems. One of the most fundamental concepts to master when learning Linux is its **file system** — the way files and directories are organized, stored, and accessed.

Unlike Windows, which uses drive letters (e.g., `C:\`, `D:\`), Linux uses a **single, unified directory tree** starting from the **root directory (`/`)**. Everything in Linux — files, directories, devices, processes, and even network sockets — is treated as a file.

---

## 2. What is a File System?

A **file system** is a method and data structure that an operating system uses to control how data is stored and retrieved. Without a file system, data placed in a storage medium would be one large body of data with no way to tell where one piece of information stops and the next begins.

Linux supports many file system types, including:

| File System | Description |
|-------------|-------------|
| **ext4** | Fourth Extended Filesystem — the default for most Linux distributions |
| **ext3** | Third Extended Filesystem — journaling file system |
| **XFS** | High-performance journaling file system, great for large files |
| **Btrfs** | B-tree file system — advanced features like snapshots and compression |
| **NTFS** | Windows file system (supported in Linux via drivers) |
| **FAT32 / exFAT** | Commonly used for USB drives and SD cards |
| **tmpfs** | Temporary file system stored in RAM |

---

## The Root Directory: `/`

```
/
```

The root directory is the top of the Linux file system tree. **Every file, directory, and device** exists under `/`. There is no directory above it.

> **Key Principle:** Only the `root` user has permission to modify contents inside this directory. Regular users will get a "Permission Denied" error if they try.

### Why This Matters

When you run out of space on `/var`, it's not a different "drive" — it's a partition or directory under the same root. Understanding the hierarchy tells you exactly where to look.

---

## Directory Reference

### `/bin` – Essential User Binaries

Contains essential command binaries that need to be available in single-user mode and to all users.

**Common commands here:**
```bash
ls, cp, mv, cat, grep, ssh, bash, pwd, echo
```

> **Note:** On modern distributions (Ubuntu, Fedora, Arch), `/bin` is often a symlink to `/usr/bin` due to the "UsrMerge" change.

---

### `/sbin` – System Administration Binaries

Similar to `/bin`, but contains binaries primarily for **system administration** and typically require root privileges.

**Common commands here:**
```bash
iptables, reboot, fdisk, ifconfig, swapon, fsck, route, init
```

> **Note:** Like `/bin`, `/sbin` is often symlinked to `/usr/sbin` on modern systems.

---

### `/boot` – Boot Loader Files

Contains files required for the **boot process**:

- Kernel image (`vmlinuz`)
- Initial RAM disk (`initrd` or `initramfs`)
- Bootloader configuration (GRUB files)

**Example files:**
```
/boot/vmlinuz-5.15.0-91-generic
/boot/initrd.img-5.15.0-91-generic
/boot/grub/grub.cfg
```

> **Tip:** `/boot` is commonly a separate small partition (500MB–1GB). It fills up when old kernels accumulate. Clean them out with your package manager.

---

### `/dev` – Device Files

Device files act as interfaces between hardware and software. Every piece of hardware is represented as a file here.

**Important device files:**

| Device | Description |
|--------|-------------|
| `/dev/sda`, `/dev/sdb` | SCSI/SATA disks |
| `/dev/nvme0n1` | NVMe drives |
| `/dev/tty` | Terminals |
| `/dev/null` | Discards anything written to it |
| `/dev/zero` | Source of null bytes |
| `/dev/random`, `/dev/urandom` | Sources of random data |
| `/dev/usbmon0` | USB monitoring |

> **Note:** `/dev` is managed by `udev` at runtime. When you plug in a USB drive, `udev` creates the appropriate device file automatically.

---

### `/etc` – Configuration Files

Contains **host-specific system-wide configuration files**. This is probably the directory you edit most as a sysadmin.

**Important files:**

| File | Purpose |
|------|---------|
| `/etc/fstab` | Filesystems to mount at boot |
| `/etc/hosts` | Static hostname to IP mappings |
| `/etc/passwd` | User account information |
| `/etc/shadow` | Encrypted user passwords |
| `/etc/resolv.conf` | DNS resolver configuration |
| `/etc/sudoers` | sudo privileges |
| `/etc/crontab` | System-wide scheduled tasks |
| `/etc/ssh/sshd_config` | SSH server configuration |

> **Tip:** Back up `/etc` regularly. A simple `rsync` of `/etc` to a remote location before making changes can save you.

---

### `/home` – User Home Directories

Personal directories for regular users. Each user gets a subdirectory:

```
/home/alice
/home/bob
/home/charlie
```

- Users can create, delete, or modify files in their own home directory
- Access to another user's home depends on security configuration
- Dotfiles live here: `~/.bashrc`, `~/.ssh/`, `~/.config/`

> **Tip:** The tilde (`~`) is shorthand for your home directory. `cd ~` and `cd /home/username` are equivalent.

---

### `/lib` & `/lib64` – Essential Shared Libraries

Contains shared libraries required by binaries in `/bin` and `/sbin`.

- `/lib` – 32-bit libraries
- `/lib64` – 64-bit libraries (on 64-bit systems)

**Example libraries:**
```
ld-2.11.1.so
libncurses.so.5.7
libc.so.6
```

> **Note:** Like `/bin`, these are typically symlinked to `/usr/lib` on modern distributions.

---

### `/mnt` & `/media` – Mount Points

| Directory | Purpose |
|-----------|---------|
| `/mnt` | Generic mount point for **manual** temporary mounts |
| `/media` | **Automatic** mount point for removable media (USB, CD, SD cards) |

**Example usage:**
```bash
# Manual mount
sudo mount /dev/sdb1 /mnt

# USB drives auto-appear here
/media/username/USB_LABEL
```

---

### `/opt` – Optional Software

Contains **add-on applications** from individual vendors — self-contained third-party software that doesn't follow the standard FHS layout.

**Example:**
```
/opt/google/chrome
/opt/teamviewer
```

> **Tip:** If you're deploying a custom application and want it cleanly isolated from system packages, `/opt` is a reasonable place.

---

### `/proc` – Process Information

A **virtual filesystem** that exists only in memory. It exposes the kernel's view of running processes and system state as files.

**Useful files:**

| File | Information |
|------|-------------|
| `/proc/cpuinfo` | CPU details |
| `/proc/meminfo` | Memory usage breakdown |
| `/proc/loadavg` | Load averages |
| `/proc/uptime` | System uptime |
| `/proc/net/dev` | Network interface statistics |
| `/proc/[PID]/` | Per-process information |

> **Key Point:** Nothing in `/proc` is stored on disk. It is regenerated every boot. Most monitoring tools (`top`, `htop`, `free`) read from `/proc`.

**Example:**
```bash
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable"
```

---

### `/root` – Root User Home

The home directory for the **root user** specifically. Kept separate from `/home` so that root always has a working home directory even if the `/home` partition is not mounted or is full.

---

### `/run` – Runtime Variable Data

A `tmpfs` mount for runtime data that needs to persist between service restarts but **not across reboots**.

- PID files
- Socket files
- Lock files
- System daemon state

> **Note:** On older systems, you may see `/var/run`, which is now typically a symlink to `/run`.

---

### `/srv` – Service Data

Contains **data served by the system**: web server document roots, FTP data, version control repositories.

**Example:**
```
/srv/http      # Web server files
/srv/ftp       # FTP server files
/srv/cvs       # CVS repositories
```

> **Note:** In practice, many admins use `/var/www` for web content, but `/srv` is the more semantically correct location per FHS.

---

### `/sys` – System Information

Another **virtual filesystem**, similar to `/proc` but more structured. It exposes the kernel's device model.

**Example usage:**
```bash
# Check I/O scheduler for a disk
cat /sys/block/sda/queue/scheduler

# CPU frequency scaling
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

---

### `/tmp` – Temporary Files

Directory for temporary files created by the system and users.

- Files are **deleted on reboot**
- World-writable (any user can write here)
- Often mounted as `tmpfs` (in RAM)

> **Note:** For temporary files that should persist across reboots, use `/var/tmp` instead.

---

### `/usr` – User Programs

The largest directory on most systems. Contains the bulk of installed software, libraries, and documentation.

**Subdirectories:**

| Directory | Contents |
|-----------|----------|
| `/usr/bin` | Most user-facing commands and applications |
| `/usr/sbin` | Non-essential system administration tools |
| `/usr/lib` | Shared libraries for `/usr/bin` and `/usr/sbin` |
| `/usr/local` | Software compiled and installed manually |
| `/usr/share` | Architecture-independent data (man pages, icons, docs) |
| `/usr/include` | Header files for C/C++ development |
| `/usr/src` | Linux kernel sources |

> **Tip:** When you compile something from source (`./configure && make && make install`), it almost always lands in `/usr/local` by default. Your package manager won't touch `/usr/local`.

---

### `/var` – Variable Data

Contains files that are expected to **change during normal system operation**: logs, mail spools, package manager databases, caches.

**Subdirectories:**

| Directory | Contents |
|-----------|----------|
| `/var/log` | System and application logs |
| `/var/lib` | Persistent application state (databases, Docker) |
| `/var/cache` | Application cache data |
| `/var/spool` | Data queued for processing (print jobs, mail) |
| `/var/tmp` | Temporary files that persist across reboots |

> **Warning:** `/var/log` filling up is a classic server problem. When `/var` runs out of space, log writes fail and applications behave strangely.

---

## Virtual File Systems

Linux uses several virtual filesystems that exist only in memory:

| Filesystem | Purpose |
|------------|---------|
| `/proc` | Process and system information |
| `/sys` | Kernel device model |
| `/dev` | Device files (managed by udev) |
| `/run` | Runtime data (tmpfs) |

These are not stored on disk — they are interfaces to kernel data structures.

---

## Linux vs Windows File System

| Feature | Linux | Windows |
|---------|-------|---------|
| Root | Single tree starting at `/` | Multiple drives (C:\, D:\) |
| Case Sensitivity | Case-sensitive (`File.txt` ≠ `file.txt`) | Case-insensitive |
| Path Separator | Forward slash `/` | Backslash `\` |
| Device Representation | Files in `/dev` | Drive letters |
| Config Files | Text files in `/etc` | Registry + config files |
| Home Directory | `/home/username` | `C:\Users\username` |

---

## Navigation Cheat Sheet

```bash
# Print current directory
pwd

# List contents
ls
ls -la      # Detailed, including hidden files

# Change directory
cd /etc
cd ~        # Go to home directory
cd ..       # Go up one level
cd -        # Go to previous directory

# Create/remove directories
mkdir newdir
rmdir emptydir
rm -r dir   # Remove directory and contents

# View files
cat file.txt
less file.txt
head -n 10 file.txt
tail -n 10 file.txt

# Search for files
find / -name "*.conf"
locate filename

# Disk usage
df -h       # Disk free (human-readable)
du -sh /var # Directory size
```

---

## Quick Reference Diagram

```
/
├── bin      → Essential user binaries (ls, cp, mv, bash)
├── sbin     → System admin binaries (fdisk, reboot, iptables)
├── boot     → Boot loader files (kernel, grub)
├── dev      → Device files (sda, tty, null)
├── etc      → Configuration files (fstab, hosts, passwd)
├── home     → User home directories (/home/alice)
├── lib      → Essential shared libraries
├── lib64    → 64-bit shared libraries
├── media    → Auto-mounted removable media (USB, CD)
├── mnt      → Manual mount points
├── opt      → Optional third-party software
├── proc     → Virtual: Process & system info
├── root     → Root user's home directory
├── run      → Runtime data (tmpfs)
├── srv      → Service data (web, ftp)
├── sys      → Virtual: Kernel device model
├── tmp      → Temporary files (cleared on reboot)
├── usr      → User programs & data
│   ├── bin  → Most user commands
│   ├── sbin → Non-essential admin tools
│   ├── lib  → Libraries for /usr/bin
│   ├── local→ Manually installed software
│   └── share→ Docs, man pages, icons
└── var      → Variable data (logs, caches, spools)
    ├── log  → System & app logs
    ├── lib  → Persistent app state
    ├── cache→ Application caches
    └── tmp  → Persistent temp files
```

---

## Sources & References

- [Filesystem Hierarchy Standard (FHS) 3.0](https://refspecs.linuxfoundation.org/fhs.shtml)
- [Linux Foundation - The Linux Filesystem Explained](https://www.linuxfoundation.org/blog/blog/classic-sysadmin-the-linux-filesystem-explained)
- [GeeksforGeeks - Linux File System](https://www.geeksforgeeks.org/linux-unix/linux-file-system/)
- [Linux Blog - Filesystem Hierarchy Explained](https://linuxblog.io/linux-filesystem-hierarchy-explained/)
- [Medium - Linux Filesystem Explained](https://medium.com/@tunacici7/linux-filesystem-explained-3307b96bb7f9)

---

> **Contributing:** Feel free to open an issue or PR if you find any inaccuracies or want to add more content!

> **License:** This guide is released under [MIT License](LICENSE).
