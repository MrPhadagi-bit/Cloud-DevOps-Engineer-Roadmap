# 🐧 Linux for Hackers — EP 1 Documentation

> **A beginner-friendly Linux reference guide based on [NetworkChuck's *Linux for Hackers* series](https://youtu.be/VbEx7B_PTOE).**  
> *Built for cybersecurity enthusiasts, ethical hackers, and anyone ready to leave GUI dependency behind.*

---

## 📋 Table of Contents

- [Why Linux?](#why-linux)
- [Getting Started](#getting-started)
- [The Terminal](#the-terminal)
- [Basic Commands](#basic-commands)
  - [Navigation](#navigation)
  - [Identity & Location](#identity--location)
  - [Finding Stuff](#finding-stuff)
  - [File & Directory Operations](#file--directory-operations)
  - [Getting Help](#getting-help)
- [The Linux Filesystem](#the-linux-filesystem)
- [Text Manipulation](#text-manipulation)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Resources & Next Steps](#resources--next-steps)

---

## 🎯 Why Linux?

Linux is the **#1 operating system for hackers** and cybersecurity professionals. Here's why:

| Reason | Description |
|--------|-------------|
| **🔓 Open Source** | The source code is freely available. You can inspect, modify, and customize every part of the OS. |
| **🔍 Transparent** | Nothing is hidden. You can see and manipulate every component — unlike closed-source systems. |
| **⚙️ Granular Control** | Infinite control via the terminal, from the smallest setting to system-wide configurations. |
| **🛠️ Hacking Tools** | Over 90% of hacking tools are written for Linux (e.g., Metasploit, Nmap, Aircrack-ng). |
| **🌐 Future-Proof** | Linux/Unix powers the internet, servers, cloud, IoT, and mobile devices. |

---

## 🚀 Getting Started

### Recommended Distributions for Hackers

| Distro | Best For | Download |
|--------|----------|----------|
| **Kali Linux** | Penetration testing & security auditing | [kali.org](https://www.kali.org) |
| **Parrot OS** | Forensics, privacy, & development | [parrotsec.org](https://www.parrotsec.org) |
| **Ubuntu** | Beginners & general-purpose Linux | [ubuntu.com](https://ubuntu.com) |

### Running Linux

- **Virtual Machine (Recommended for beginners):** Use VirtualBox or VMware to run Linux inside your current OS safely.
- **Dual Boot:** Install Linux alongside your main OS.
- **Live USB:** Boot Linux from a USB stick without installing.
- **WSL (Windows Subsystem for Linux):** Run Linux natively on Windows 10/11.

---

## 💻 The Terminal

The **terminal** (or shell) is your command-line interface to Linux. It's where the magic happens.

### Common Shells

| Shell | Description |
|-------|-------------|
| `bash` | Bourne Again Shell — the default on most distros |
| `zsh` | Z Shell — enhanced features, popular among power users |
| `fish` | Friendly Interactive Shell — user-friendly with autosuggestions |

> 💡 **Pro Tip:** The terminal is faster, more powerful, and more scriptable than any GUI. Master it.

---

## ⌨️ Basic Commands

### Navigation

| Command | Description | Example |
|---------|-------------|---------|
| `pwd` | **P**rint **W**orking **D**irectory — shows your current location | `pwd` |
| `cd` | **C**hange **D**irectory — move to another folder | `cd /home/user` |
| `cd ~` | Go to your home directory | `cd ~` |
| `cd ..` | Go up one directory level | `cd ..` |
| `cd -` | Go back to the previous directory | `cd -` |
| `ls` | **L**i**s**t files and directories | `ls` |
| `ls -la` | List all files (including hidden) with details | `ls -la` |
| `ls -lh` | List files with human-readable sizes | `ls -lh` |

```bash
# Example navigation session
pwd                          # /home/kali
cd /etc                      # move to /etc
pwd                          # /etc
cd ..                        # move up to /
cd ~                         # back to home
ls -la                       # list everything in home
```

---

### Identity & Location

| Command | Description | Example |
|---------|-------------|---------|
| `whoami` | Display the current username | `whoami` |
| `id` | Show user ID and group info | `id` |
| `hostname` | Display the system hostname | `hostname` |
| `uname -a` | Show system kernel info | `uname -a` |

```bash
whoami                       # kali
id                           # uid=1000(kali) gid=1000(kali) groups=1000(kali)
hostname                     # kali-vm
uname -a                     # Linux kali-vm 5.18.0-kali5-amd64 #1 SMP ...
```

---

### Finding Stuff

| Command | Description | Example |
|---------|-------------|---------|
| `find` | Search for files/directories | `find / -name "*.txt"` |
| `locate` | Quickly find files by name (uses a database) | `locate passwd` |
| `which` | Show the path of an executable | `which python3` |
| `whereis` | Locate binary, source, and man pages | `whereis nmap` |
| `grep` | **G**lobally search a **R**egular **E**xpression and **P**rint | `grep "root" /etc/passwd` |

```bash
# Find all .conf files under /etc
find /etc -name "*.conf"

# Find files modified in the last 24 hours
find /home -mtime -1

# Search for the word "password" in a file
grep "password" notes.txt

# Case-insensitive search
grep -i "error" logfile.txt

# Recursive search in a directory
grep -r "TODO" /home/user/projects/

# Show line numbers
grep -n "root" /etc/passwd
```

---

### File & Directory Operations

| Command | Description | Example |
|---------|-------------|---------|
| `touch` | Create an empty file | `touch notes.txt` |
| `mkdir` | **M**a**k**e **Dir**ectory | `mkdir projects` |
| `mkdir -p` | Create nested directories | `mkdir -p projects/linux/ep1` |
| `cp` | **C**o**p**y files/directories | `cp file.txt backup.txt` |
| `cp -r` | Copy directories recursively | `cp -r folder/ backup/` |
| `mv` | **M**o**v**e or rename files | `mv old.txt new.txt` |
| `rm` | **R**e**m**ove files | `rm file.txt` |
| `rm -r` | Remove directories recursively | `rm -r folder/` |
| `rm -rf` | **Force** remove (⚠️ DANGEROUS) | `rm -rf folder/` |
| `cat` | Con**cat**enate and display file contents | `cat file.txt` |
| `head` | Show first 10 lines of a file | `head log.txt` |
| `tail` | Show last 10 lines of a file | `tail log.txt` |
| `tail -f` | Follow a file in real-time | `tail -f /var/log/syslog` |
| `less` | View file contents interactively | `less largefile.txt` |
| `more` | View file contents page by page | `more file.txt` |

```bash
# Create a file and add content
touch hack.txt
echo "Hello, hacker!" > hack.txt

# View the file
cat hack.txt

# Copy it
cp hack.txt hack_backup.txt

# Rename it
mv hack.txt hello.txt

# Create a directory structure
mkdir -p ~/hacking/tools/recon

# Remove a file
rm hello.txt

# View system logs (needs sudo)
sudo tail -f /var/log/auth.log
```

---

### Getting Help

| Command | Description | Example |
|---------|-------------|---------|
| `man` | **Man**ual pages — the built-in documentation | `man ls` |
| `man -k` | Search man pages by keyword | `man -k password` |
| `whatis` | Brief description of a command | `whatis grep` |
| `--help` | Show brief usage info | `ls --help` |
| `info` | Detailed info documentation | `info coreutils` |

```bash
# Read the manual for the 'find' command
man find

# Search for commands related to "network"
man -k network

# Quick description of 'chmod'
whatis chmod

# Quick help for 'grep'
grep --help
```

> 📖 **Pro Tip:** Inside `man` pages, press `q` to quit, `/` to search, and `Space` to scroll down.

---

## 🗂️ The Linux Filesystem

Everything in Linux is a **file**. The filesystem is organized in a hierarchical tree structure starting from the root `/`.

```
/
├── bin      → Essential user binaries (ls, cat, cp, etc.)
├── sbin     → Essential system binaries (fdisk, reboot, etc.)
├── usr      → User programs, libraries, documentation
│   ├── bin  → Non-essential user binaries
│   └── sbin → Non-essential system binaries
├── boot     → Boot loader files
├── dev      → Device files (disks, terminals, etc.)
├── etc      → Configuration files
├── home     → User home directories
│   └── kali → Your personal files
├── root     → Root user's home directory
├── lib      → Shared libraries
├── tmp      → Temporary files (cleared on reboot)
├── var      → Variable data (logs, caches, mail)
├── mnt      → Temporary mount points
└── media    → Removable media mount points
```

### Key Directories Explained

| Directory | Purpose |
|-----------|---------|
| `/bin` | Essential commands available to all users |
| `/sbin` | System administration commands |
| `/usr` | Secondary hierarchy for user data and applications |
| `/etc` | System-wide configuration files |
| `/home` | Personal directories for regular users |
| `/root` | Home directory for the root (superuser) account |
| `/dev` | Device files representing hardware |
| `/var` | Variable data like logs (`/var/log`) and spools |
| `/tmp` | Temporary files — anyone can write here |
| `/mnt` & `/media` | Mount points for external drives and USBs |
| `/lib` | Essential shared libraries needed by binaries |
| `/boot` | Files needed to boot the system |

---

## 📝 Text Manipulation

Linux is built around text. Learning to manipulate it is crucial.

| Command | Description | Example |
|---------|-------------|---------|
| `echo` | Print text to stdout | `echo "Hello World"` |
| `>` | Redirect output to a file (overwrite) | `echo "hi" > file.txt` |
| `>>` | Redirect output to a file (append) | `echo "bye" >> file.txt` |
| `|` | Pipe — send output to another command | `cat file.txt \| grep "error"` |
| `sort` | Sort lines alphabetically | `sort names.txt` |
| `uniq` | Filter duplicate lines | `sort names.txt \| uniq` |
| `wc` | **W**ord **C**ount — lines, words, bytes | `wc -l file.txt` |
| `cut` | Extract sections from lines | `cut -d':' -f1 /etc/passwd` |
| `sed` | **S**tream **Ed**itor — find and replace | `sed 's/old/new/g' file.txt` |
| `awk` | Pattern scanning and processing | `awk '{print $1}' file.txt` |

```bash
# Count lines in a file
wc -l /etc/passwd

# Extract usernames from /etc/passwd
cut -d':' -f1 /etc/passwd

# Find and replace text
sed 's/kali/admin/g' users.txt

# Chain commands with pipes
cat /etc/passwd | grep "/bin/bash" | cut -d':' -f1

# Sort and remove duplicates
cat words.txt | sort | uniq
```

---

## 🧠 Quick Reference Cheat Sheet

### Navigation & Basics
```bash
pwd              # Where am I?
cd <dir>         # Go to directory
ls -la           # List all files
clear            # Clear the screen
history          # Show command history
```

### Files & Directories
```bash
touch <file>     # Create empty file
mkdir <dir>      # Create directory
cp <src> <dst>   # Copy
mv <src> <dst>   # Move/Rename
rm <file>        # Remove file
rm -r <dir>      # Remove directory
cat <file>       # View file contents
head/tail <file> # View start/end of file
less <file>      # Interactive file viewer
```

### Finding & Searching
```bash
find <path> -name "<pattern>"
locate <name>
which <command>
whereis <command>
grep "<pattern>" <file>
```

### Help
```bash
man <command>
<command> --help
whatis <command>
```

### Superuser (Root)
```bash
sudo <command>   # Run command as root
sudo -i          # Switch to root user
su -             # Switch to root (needs password)
```

---

## 📚 Resources & Next Steps

### Official Resources
- 📺 [Linux for Hackers — Full Series (NetworkChuck)](https://www.youtube.com/playlist?list=PLIhvC56v63IL2OjFvv_PI0B2yAXGfJLMI)
- 🎓 [Hack The Box Academy — Linux Fundamentals](https://academy.hackthebox.com/)
- 📖 [Linux Basics for Hackers (Book by OccupyTheWeb)](https://www.amazon.com/Linux-Basics-Hackers-Networking-Scripting/dp/1593278551)

### Practice Platforms
| Platform | Description |
|----------|-------------|
| [Hack The Box](https://www.hackthebox.com) | Penetration testing labs |
| [TryHackMe](https://tryhackme.com) | Guided cybersecurity training |
| [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) | Linux command-line wargames |
| [Linux Journey](https://linuxjourney.com) | Free interactive Linux lessons |

### What to Learn Next
1. 🔐 **File Permissions** — `chmod`, `chown`, `ls -la`
2. 🧑‍💻 **Process Management** — `ps`, `top`, `kill`, `nice`
3. 🌐 **Networking** — `ifconfig`/`ip`, `ping`, `netstat`, `nmap`
4. 📦 **Package Management** — `apt`, `dpkg`, `snap`
5. 🐚 **Bash Scripting** — Automate your workflow
6. 🔧 **Services** — `systemctl`, `service`, cron jobs

---

## 🤝 Contributing

Found an error or want to add more commands? PRs are welcome! This doc is meant to grow with the community.

---

> ☕ *"Coffee + Linux = Hacker Fuel"* — NetworkChuck

**License:** MIT | **Maintained by:** [Your Name] | **Based on:** NetworkChuck's Linux for Hackers Series
