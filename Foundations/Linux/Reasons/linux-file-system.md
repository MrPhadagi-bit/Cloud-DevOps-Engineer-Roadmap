# Linux File System — Complete Guide

> A comprehensive, beginner-friendly guide to understanding the Linux File System Hierarchy Standard (FHS), directory structure, navigation, and essential command-line operations.

---


## 1. Introduction

Linux is an **open-source, Unix-like operating system** that powers everything from smartphones and laptops to servers, supercomputers, and embedded systems. One of the most fundamental concepts to master when learning Linux is its **file system** — the way files and directories are organized, stored, and accessed.

Unlike Windows, which uses drive letters (e.g., `C:\`, `D:\`), Linux uses a **single, unified directory tree** starting from the **root directory (`/`)**. Everything in Linux — files, directories, devices, processes, and even network sockets — is treated as a file.

> **Key Principle:** In Linux, *everything is a file*.

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

You can check your current file system type using:

```bash
df -T
```

---

## 3. File System Hierarchy Standard (FHS)

The **File System Hierarchy Standard (FHS)** defines the directory structure and directory contents in Linux distributions. It was created to provide consistency across different Linux distributions, making it easier for users, software developers, and system administrators to understand where files should be located.

> **Why FHS matters:** It ensures that no matter which Linux distribution you use (Ubuntu, Fedora, Debian, Arch, etc.), the core directory structure remains predictable and familiar.

---

## 4. The Linux Directory Structure

Here is a visual overview of the Linux directory tree:

```
/
├── bin
├── boot
├── dev
├── etc
├── home
│   ├── alice
│   ├── bob
│   └── charlie
├── lib
├── lib64
├── lost+found
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
│   ├── bin
│   ├── include
│   ├── lib
│   ├── local
│   ├── sbin
│   └── share
└── var
    ├── cache
    ├── log
    ├── mail
    ├── spool
    └── tmp
```

---

### 4.1 Root Directory (`/`)

The **root directory** is the top-level directory in the Linux file system hierarchy. All other directories branch out from here. It is represented by a single forward slash (`/`).

> **Important:** Do not confuse the root directory `/` with the `/root` directory. `/` is the top of the file system, while `/root` is the home directory of the root (superuser) account.

---

### 4.2 `/bin` — Essential User Binaries

The `/bin` directory contains **essential command binaries** that are available to all users. These are the commands you need in single-user mode (e.g., for system repair) and are required for the system to boot and operate in normal mode.

**Common commands found here:**

| Command | Description |
|---------|-------------|
| `cat` | Concatenate and display file content |
| `cp` | Copy files and directories |
| `ls` | List directory contents |
| `mv` | Move or rename files |
| `rm` | Remove files |
| `mkdir` | Make directories |
| `echo` | Display a line of text |
| `pwd` | Print working directory |

```bash
ls /bin
```

---

### 4.3 `/sbin` — System Binaries

The `/sbin` directory contains **system administration binaries** — commands used for system maintenance and administration tasks. These commands are typically used by the root user.

**Common commands found here:**

| Command | Description |
|---------|-------------|
| `fdisk` | Partition table manipulator |
| `fsck` | File system consistency check |
| `ifconfig` / `ip` | Network interface configuration |
| `reboot` | Reboot the system |
| `shutdown` | Shut down the system |
| `mkfs` | Build a Linux file system |

```bash
ls /sbin
```

---

### 4.4 `/etc` — Configuration Files

The `/etc` directory contains **system-wide configuration files** and shell scripts used to boot and initialize the system. This is one of the most important directories for system administrators.

**Important files and subdirectories:**

| Path | Description |
|------|-------------|
| `/etc/passwd` | User account information |
| `/etc/shadow` | Secure user account information (passwords) |
| `/etc/group` | Group account information |
| `/etc/hosts` | Static table lookup for hostnames |
| `/etc/resolv.conf` | DNS resolver configuration |
| `/etc/fstab` | File system table (mount points at boot) |
| `/etc/ssh/` | SSH server configuration |
| `/etc/cron.d/` | Cron job configurations |
| `/etc/nginx/` | Nginx web server configuration |
| `/etc/apt/` | APT package manager configuration (Debian/Ubuntu) |

```bash
ls /etc
```

---

### 4.5 `/dev` — Device Files

In Linux, **everything is a file** — including hardware devices. The `/dev` directory contains special files that represent devices attached to the system. These are not actual files on disk but interfaces to device drivers.

**Common device files:**

| Device | Description |
|--------|-------------|
| `/dev/sda` | First SCSI/SATA hard disk |
| `/dev/sda1` | First partition on the first hard disk |
| `/dev/tty` | Terminal devices |
| `/dev/null` | Null device (discards all input) |
| `/dev/zero` | Produces a continuous stream of zeros |
| `/dev/random` | Random number generator |
| `/dev/urandom` | Non-blocking random number generator |
| `/dev/stdin` | Standard input |
| `/dev/stdout` | Standard output |
| `/dev/stderr` | Standard error |

```bash
ls /dev
```

---

### 4.6 `/proc` — Process Information

The `/proc` directory is a **virtual file system** that provides information about running processes and the kernel. It does not exist on disk — it is created dynamically in memory when the system boots.

**Key files and directories:**

| Path | Description |
|------|-------------|
| `/proc/cpuinfo` | CPU information |
| `/proc/meminfo` | Memory usage statistics |
| `/proc/uptime` | System uptime |
| `/proc/version` | Kernel version |
| `/proc/[PID]/` | Directory for each running process (PID = Process ID) |
| `/proc/filesystems` | Supported file systems |
| `/proc/mounts` | Currently mounted file systems |

```bash
cat /proc/cpuinfo
cat /proc/meminfo
```

---

### 4.7 `/var` — Variable Data Files

The `/var` directory contains **variable data files** — files whose content is expected to change during normal system operation. This includes logs, caches, spool files, and temporary files that need to persist between reboots.

**Important subdirectories:**

| Path | Description |
|------|-------------|
| `/var/log/` | System log files |
| `/var/log/syslog` | General system activity log |
| `/var/log/auth.log` | Authentication log |
| `/var/mail/` | User mailboxes |
| `/var/spool/` | Spool directories (print queues, cron jobs) |
| `/var/cache/` | Application cache data |
| `/var/tmp/` | Temporary files that persist across reboots |
| `/var/www/` | Web server document root (common convention) |

```bash
ls /var/log
```

---

### 4.8 `/tmp` — Temporary Files

The `/tmp` directory is used for **temporary files** created by applications and users. Files in `/tmp` are typically deleted on system reboot (though this depends on the distribution's configuration).

> **Note:** Any user can write to `/tmp`, making it useful for shared temporary storage.

```bash
ls /tmp
```

---

### 4.9 `/usr` — User Programs

The `/usr` directory contains **user programs, libraries, documentation, and source code** for non-essential system programs. It is the largest directory on most Linux systems.

**Subdirectories:**

| Path | Description |
|------|-------------|
| `/usr/bin/` | Non-essential user binaries |
| `/usr/sbin/` | Non-essential system administration binaries |
| `/usr/lib/` | Libraries for `/usr/bin` and `/usr/sbin` |
| `/usr/local/` | Locally installed software (not managed by package manager) |
| `/usr/share/` | Architecture-independent data (documentation, icons, fonts) |
| `/usr/include/` | Header files for C/C++ compilation |
| `/usr/src/` | Source code |

> **Modern distinction:** `/bin` and `/sbin` contain essential binaries needed at boot; `/usr/bin` and `/usr/sbin` contain the rest.

```bash
ls /usr/bin | wc -l   # See how many commands are available!
```

---

### 4.10 `/home` — Home Directories

The `/home` directory contains **personal directories** for regular users. Each user gets their own subdirectory (e.g., `/home/alice`, `/home/bob`) where they can store personal files, configurations, and documents.

> **Best Practice:** Always work in your home directory or `/tmp` for personal projects. Avoid modifying system directories unless you know what you're doing.

```bash
cd ~              # Go to your home directory
cd /home/username # Go to a specific user's home
pwd               # Print current directory
```

---

### 4.11 `/boot` — Boot Loader Files

The `/boot` directory contains files needed to **boot the Linux system**, including the Linux kernel, initial RAM disk (initrd), and boot loader configuration files (GRUB).

**Key files:**

| File | Description |
|------|-------------|
| `vmlinuz` | The compressed Linux kernel |
| `initrd.img` | Initial RAM disk image |
| `grub/` | GRUB boot loader configuration |

> **Warning:** Be very careful when modifying files in `/boot`. A mistake here can prevent the system from booting.

```bash
ls /boot
```

---

### 4.12 `/lib` — Essential Shared Libraries

The `/lib` directory contains **shared library files** needed by the essential binaries in `/bin` and `/sbin`. These are similar to `.dll` files in Windows.

On 64-bit systems, you will also see `/lib64`, which contains 64-bit versions of these libraries.

```bash
ls /lib
ls /lib64
```

---

### 4.13 `/opt` — Optional Add-on Software

The `/opt` directory is used for **optional or third-party software** that is not part of the default installation. Commercial software packages often install here.

```bash
ls /opt
```

---

### 4.14 `/mnt` & `/media` — Mount Points

These directories are used for **mounting external file systems** (USB drives, CD-ROMs, network shares, etc.).

| Directory | Usage |
|-----------|-------|
| `/media` | Automatically mounted removable media (USB drives, CDs, DVDs) |
| `/mnt` | Temporary mount points for system administrators |

```bash
ls /media
ls /mnt
```

---

### 4.15 `/srv` — Service Data

The `/srv` directory contains **data for services** provided by the system, such as web servers, FTP servers, or version control systems.

```bash
ls /srv
```

---

### 4.16 `/lost+found` — Recovered Files

The `/lost+found` directory is created by the `fsck` utility during file system checks. It contains **recovered files** that were found to be corrupted or disconnected from the directory structure after a system crash or improper shutdown.

> **Note:** You typically don't need to interact with this directory unless you're performing data recovery.

---

## 5. File Types in Linux

Linux recognizes several file types, which can be identified using the `ls -l` command:

| Symbol | File Type | Description |
|--------|-----------|-------------|
| `-` | Regular file | Text files, binary files, images, etc. |
| `d` | Directory | A folder containing files and other directories |
| `l` | Symbolic link | A pointer to another file |
| `c` | Character device | Devices that handle data character by character (e.g., terminals) |
| `b` | Block device | Devices that handle data in blocks (e.g., hard drives) |
| `s` | Socket | Inter-process communication endpoint |
| `p` | Named pipe (FIFO) | Allows processes to communicate |

```bash
ls -l /dev
# Look at the first character of each line
```

---

## 6. Navigating the File System

### 6.1 Absolute vs. Relative Paths

- **Absolute Path:** Starts from the root directory (`/`). Always points to the same location regardless of your current directory.
  ```bash
  /home/alice/documents/report.txt
  /etc/ssh/sshd_config
  ```

- **Relative Path:** Starts from your current directory. Changes based on where you are.
  ```bash
  ./documents/report.txt    # In the current directory
  ../backup/config.txt      # One directory up
  ../../shared/file.txt     # Two directories up
  ```

**Special directory references:**

| Symbol | Meaning |
|--------|---------|
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Home directory of current user |
| `~username` | Home directory of specified user |
| `-` | Previous directory (useful with `cd -`) |

---

### 6.2 Essential Navigation Commands

| Command | Description | Example |
|---------|-------------|---------|
| `pwd` | Print working directory | `pwd` |
| `cd` | Change directory | `cd /var/log` |
| `ls` | List directory contents | `ls -la /etc` |
| `tree` | Display directory tree | `tree -L 2 /usr` |
| `find` | Search for files | `find /home -name "*.txt"` |
| `locate` | Find files by name (uses database) | `locate nginx.conf` |
| `which` | Locate a command | `which python3` |
| `whereis` | Locate binary, source, and man page | `whereis ls` |

**`ls` command options:**

```bash
ls              # Basic listing
ls -l           # Long format (permissions, owner, size, date)
ls -a           # Show hidden files (files starting with .)
ls -la          # Combine long format and hidden files
ls -lh          # Human-readable file sizes (KB, MB, GB)
ls -ltr         # Sort by time, reversed (oldest last)
ls --color=auto # Color-coded output
```

---

## 7. File & Directory Operations

| Command | Description | Example |
|---------|-------------|---------|
| `touch` | Create an empty file or update timestamp | `touch newfile.txt` |
| `mkdir` | Create a directory | `mkdir mydir` |
| `mkdir -p` | Create nested directories | `mkdir -p a/b/c` |
| `cp` | Copy files or directories | `cp file.txt backup/` |
| `cp -r` | Copy directories recursively | `cp -r dir1/ dir2/` |
| `mv` | Move or rename files/directories | `mv old.txt new.txt` |
| `rm` | Remove files | `rm file.txt` |
| `rm -r` | Remove directories recursively | `rm -r mydir/` |
| `rm -rf` | Force remove (⚠️ dangerous!) | `rm -rf mydir/` |
| `rmdir` | Remove empty directories | `rmdir emptydir/` |
| `cat` | Display file content | `cat file.txt` |
| `less` | View file content (scrollable) | `less /var/log/syslog` |
| `head` | Show first 10 lines | `head file.txt` |
| `tail` | Show last 10 lines | `tail -f /var/log/syslog` |
| `ln` | Create a hard link | `ln file.txt link.txt` |
| `ln -s` | Create a symbolic (soft) link | `ln -s /path/to/file linkname` |

> **⚠️ Warning:** `rm -rf /` will delete your entire file system. Be extremely careful with recursive and force flags!

---

## 8. Understanding File Permissions

Linux uses a **permission system** to control who can read, write, or execute files and directories.

### 8.1 Permission Categories

Permissions are divided into three categories:

| Category | Symbol | Description |
|----------|--------|-------------|
| **Owner** | `u` | The user who owns the file |
| **Group** | `g` | Users belonging to the file's group |
| **Others** | `o` | Everyone else |

Each category has three permission types:

| Permission | Symbol | Value | Description |
|------------|--------|-------|-------------|
| **Read** | `r` | 4 | View file contents / list directory |
| **Write** | `w` | 2 | Modify file / create/delete in directory |
| **Execute** | `x` | 1 | Run file as program / enter directory |

**Example:**

```bash
ls -l /etc/passwd
# Output: -rw-r--r-- 1 root root 2875 Aug 10 09:00 /etc/passwd
```

Breaking down `-rw-r--r--`:

```
-  rw-  r--  r--
│   │    │    │
│   │    │    └── Others: read only (r-- = 4)
│   │    └─────── Group: read only (r-- = 4)
│   └──────────── Owner: read + write (rw- = 6)
└──────────────── File type: regular file (-)
```

So the numeric permission is **644**.

**Common permission patterns:**

| Numeric | Symbolic | Meaning |
|---------|----------|---------|
| `777` | `rwxrwxrwx` | Full access for everyone (⚠️ insecure) |
| `755` | `rwxr-xr-x` | Owner full, others read+execute (common for scripts) |
| `644` | `rw-r--r--` | Owner read+write, others read only (common for files) |
| `700` | `rwx------` | Owner only (private files) |
| `600` | `rw-------` | Owner read+write only (private data files) |

---

### 8.2 Changing Permissions

Use the `chmod` command to change permissions:

**Symbolic mode:**

```bash
chmod u+x script.sh      # Add execute for owner
chmod g-w file.txt       # Remove write for group
chmod o=r file.txt       # Set others to read only
chmod a+x script.sh      # Add execute for all (a = all)
chmod u=rwx,g=rx,o=rx dir/  # Set multiple at once
```

**Numeric mode:**

```bash
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 700 private.key    # rwx------
```

---

### 8.3 Ownership

Use `chown` to change the owner and `chgrp` to change the group:

```bash
chown alice file.txt           # Change owner to alice
chown alice:developers file.txt # Change owner and group
chown -R alice:alice /home/alice/projects  # Recursive change
chgrp developers file.txt      # Change group only
```

---

## 9. Hidden Files & Directories

In Linux, any file or directory that starts with a **dot (`.`)** is considered hidden. Hidden files are not shown by default when listing directories.

**Common hidden files:**

| File | Description |
|------|-------------|
| `.bashrc` | Bash shell configuration |
| `.bash_profile` | Bash login configuration |
| `.profile` | User profile settings |
| `.ssh/` | SSH keys and configuration |
| `.config/` | Application configurations (XDG standard) |
| `.local/` | User-specific local data |
| `.cache/` | User-specific cache |
| `.vimrc` | Vim editor configuration |
| `.gitignore` | Git ignore rules |

```bash
ls -la ~          # List all files including hidden ones in home
cd ~
ls -la            # See your hidden configuration files
```

---

## 10. Pipes & Redirection

Linux allows you to redirect input and output streams, and chain commands together using pipes.

**Standard streams:**

| Stream | File Descriptor | Description |
|--------|-----------------|-------------|
| **stdin** | 0 | Standard input (keyboard) |
| **stdout** | 1 | Standard output (screen) |
| **stderr** | 2 | Standard error (screen) |

**Redirection operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `>` | Redirect stdout to file (overwrite) | `ls > files.txt` |
| `>>` | Redirect stdout to file (append) | `echo "log" >> log.txt` |
| `<` | Redirect file to stdin | `sort < unsorted.txt` |
| `2>` | Redirect stderr | `command 2> errors.txt` |
| `&>` | Redirect stdout and stderr | `command &> output.txt` |
| `\|` | Pipe stdout to another command | `cat file \| grep "error"` |

**Examples:**

```bash
# Save command output to a file
ls -la /etc > etc_files.txt

# Append to a file
echo "New line" >> notes.txt

# Pipe: chain commands together
ps aux | grep nginx

# Count lines in a file
wc -l /etc/passwd

# Find and count
find /var/log -name "*.log" | wc -l

# Filter and sort
cat /etc/passwd | grep "/bin/bash" | sort

# Redirect errors to /dev/null (discard them)
find / -name "secret.txt" 2>/dev/null

# Redirect both stdout and stderr
make build &> build.log
```

---

## 11. Practical Examples

### Example 1: Exploring Your System

```bash
# Check disk usage of directories
du -sh /*

# Find the largest directories
du -sh /var/* | sort -rh | head -10

# Check available disk space
df -h

# See what file systems are mounted
mount | column -t
```

### Example 2: Finding Files

```bash
# Find all .log files modified in the last 7 days
find /var/log -name "*.log" -mtime -7

# Find files larger than 100MB
find /home -size +100M

# Find empty directories
find /tmp -type d -empty

# Search for a file by name
find / -name "nginx.conf" 2>/dev/null

# Use locate (requires updatedb)
updatedb
locate *.conf | grep nginx
```

### Example 3: Working with Permissions

```bash
# Create a script and make it executable
touch backup.sh
chmod +x backup.sh
ls -l backup.sh
# -rwxr-xr-x 1 user group ...

# Create a private directory
mkdir private_stuff
chmod 700 private_stuff
ls -ld private_stuff
# drwx------ 2 user group ...

# Fix permissions recursively
chmod -R 755 ~/scripts
chown -R $(whoami):$(whoami) ~/projects
```

### Example 4: Monitoring System

```bash
# Watch a log file in real-time
tail -f /var/log/syslog

# Check running processes
ps aux | less

# Check memory usage
free -h
cat /proc/meminfo | grep Mem

# Check CPU info
cat /proc/cpuinfo | grep "model name"

# Check system uptime
uptime
cat /proc/uptime
```

---

## 12. Cheat Sheet

### Directory Quick Reference

| Directory | Purpose |
|-----------|---------|
| `/` | Root directory — top of the hierarchy |
| `/bin` | Essential user binaries |
| `/sbin` | Essential system binaries |
| `/etc` | System-wide configuration files |
| `/dev` | Device files |
| `/proc` | Process and kernel information (virtual) |
| `/var` | Variable data (logs, caches, mail) |
| `/tmp` | Temporary files (cleared on reboot) |
| `/usr` | User programs and libraries |
| `/home` | User home directories |
| `/boot` | Boot loader files |
| `/lib` | Essential shared libraries |
| `/opt` | Optional third-party software |
| `/mnt` | Temporary mount points |
| `/media` | Removable media mount points |
| `/root` | Root user's home directory |
| `/srv` | Service data |
| `/sys` | System information (virtual) |
| `/run` | Runtime variable data |

### Command Quick Reference

| Command | Description |
|---------|-------------|
| `pwd` | Print working directory |
| `cd ~` | Go to home directory |
| `cd -` | Go to previous directory |
| `ls -la` | List all files with details |
| `ls -lh` | List with human-readable sizes |
| `mkdir -p` | Create nested directories |
| `cp -r` | Copy directories recursively |
| `rm -rf` | Remove forcefully (⚠️ use with caution) |
| `cat` | View file contents |
| `less` | Scrollable file viewer |
| `head -n 20` | Show first 20 lines |
| `tail -f` | Follow file changes in real-time |
| `chmod 755` | Set permissions to rwxr-xr-x |
| `chown user:group` | Change owner and group |
| `find / -name` | Search for files by name |
| `du -sh` | Show directory size |
| `df -h` | Show disk space usage |
| `ps aux` | List all running processes |
| `grep` | Search text patterns |
| `|` | Pipe output to another command |
| `>` | Redirect output to file |
| `>>` | Append output to file |

---

## 13. References

- [File System Hierarchy Standard (FHS) — Wikipedia](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)
- [Linux Foundation — FHS Documentation](https://refspecs.linuxfoundation.org/fhs.shtml)
- [Linux File System for Beginners — Digital Cloud Training (YouTube)](https://youtu.be/VyLnhf7BS-c)
- [The Linux Documentation Project](https://tldp.org/)
- [Linux man pages](https://man7.org/linux/man-pages/)

---

> **Contributing:** Feel free to open issues or submit pull requests if you find any inaccuracies or want to add more examples!
>
> **License:** This guide is provided for educational purposes. Share freely and learn together.
