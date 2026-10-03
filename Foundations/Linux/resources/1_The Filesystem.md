# 1. The Filesystem

## Introduction: Everything Is a File

In Linux, **everything is treated as a file** — even hardware devices, sockets, and processes. These files and folders are arranged in a **single hierarchical structure** that starts at the root directory `/`.

Imagine the Linux filesystem like a **family tree**:

- `/` is the root — the top-most "parent" directory.
- Every other directory and file is a "child" or "descendant" of `/`.

> **Key difference from Windows:** This structure is very different from Windows, where you might have `C:\`, `D:\`, and so on. In Linux, it's all **one unified structure**.

---

## Understanding Key Directories (Beginner's Guide)

| Directory | What It Contains | Real Use Cases |
|-----------|------------------|----------------|
| `/` | Root directory. Everything starts here. | The base of the entire system. |
| `/home/` | User folders (`/home/nimesha`, etc.) | Store personal files, scripts, downloads. |
| `/etc/` | Configuration files | Configure services like Nginx, SSH, cron, firewall, etc. |
| `/var/` | Variable data: logs, mail, spools | Monitor logs (`/var/log`), troubleshoot crashes. |
| `/usr/` | User-installed software and system-wide resources | Holds app binaries and libraries. |
| `/bin/` and `/sbin/` | Essential system binaries and system commands | Contains tools like `ls`, `cp`, `shutdown`. |
| `/tmp/` | Temporary files (deleted on reboot) | Store transient files like downloaded archives. |
| `/opt/` | Optional third-party software | Install tools like Anaconda, or custom server software. |
| `/dev/` | Device files for hardware (like USB, disks) | Interact with drives and peripherals. |
| `/proc/` and `/sys/` | Kernel and process info presented as virtual files | Useful for system monitoring and diagnostics. |

---

## Real-World Example: Configuring a Web Server

Let's say you're deploying a web server using **Nginx**. Here's where you'll interact with the filesystem:

**1. View Nginx config:**

```bash
cd /etc/nginx/
ls -l
nano nginx.conf
```

**2. Serve HTML from:**

```bash
cd /var/www/html/
```

**3. Check logs:**

```bash
tail -f /var/log/nginx/access.log
```

> Knowing where these files live saves you hours of guessing and lets you troubleshoot confidently.

---

## How to Explore the Filesystem

**Where am I?**

```bash
pwd
```

**What's here?**

```bash
ls -l
```

**Move around:**

```bash
cd /etc
cd ~/Downloads
cd ..
```

### Tips

- `..` moves you **up one directory level**.
- `~` brings you back to your **home directory**.
- **Tab key = auto-complete paths. Use it always!**

---

## Clean Structure = Cleaner Scripts

When writing Bash scripts or automation code, referencing the correct path is key:

```bash
cp /etc/nginx/nginx.conf /home/nimesha/backup/
```

> Misplacing even a single `/` can break your automation.

---

## Best Practices for Beginners

1. **Don't delete or modify** anything in `/etc` or `/bin` unless you know what it does.
2. **Keep your personal work** (scripts, labs, etc.) in `/home/yourname/`.
3. **Create a `/home/yourname/projects/` folder** to separate each task or project.
4. **Back up important configs before editing:**

   ```bash
   cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   ```
