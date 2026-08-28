# 🖥️ Linux Processes: `top` & `htop` Explained

> **Reference Video:** [Linux Processes, top & htop Tutorial](https://youtu.be/nQhRRLgLFaQ?si=MY1wozAiMJMIEY5p)  
> A comprehensive guide to understanding and managing Linux processes using `top` and `htop`.

---

## 📑 Table of Contents

- [What is a Process?](#what-is-a-process)
- [Why Monitor Processes?](#why-monitor-processes)
- [The `top` Command](#the-top-command)
  - [Understanding the `top` Header](#understanding-the-top-header)
  - [Process List Columns](#process-list-columns)
  - [Interactive Commands in `top`](#interactive-commands-in-top)
  - [Batch Mode](#batch-mode)
- [The `htop` Command](#the-htop-command)
  - [Installing `htop`](#installing-htop)
  - [Understanding the `htop` Interface](#understanding-the-htop-interface)
  - [Color Coding](#color-coding)
  - [Interactive Commands in `htop`](#interactive-commands-in-htop)
  - [Command-Line Options](#command-line-options)
- [Process States Explained](#process-states-explained)
- [Memory Metrics: VIRT, RES, SHR](#memory-metrics-virt-res-shr)
- [Load Average Explained](#load-average-explained)
- [`top` vs `htop`: Comparison](#top-vs-htop-comparison)
- [Practical Workflows](#practical-workflows)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## What is a Process?

In Linux, a **process** is an instance of a running program. Every command, service, daemon, or application you launch creates one or more processes. Each process is assigned a unique identifier called a **PID (Process ID)**.

Key concepts:
- **PID** – Unique Process ID number
- **PPID** – Parent Process ID (the process that spawned it)
- **Threads** – Lightweight units of execution within a process
- **Kernel Threads** – Threads managed by the Linux kernel itself

Processes form a **tree structure**: when a process launches another, it becomes the parent. The root of this tree is `init` or `systemd` (PID 1).

---

## Why Monitor Processes?

Monitoring processes is essential for:
- 🔍 **Troubleshooting** slow or unresponsive systems
- 🧠 **Identifying** memory or CPU-heavy applications
- 🧹 **Killing** unresponsive or runaway processes
- 🛡️ **Monitoring** server performance in real-time
- 📊 **Understanding** system resource utilization

---

## The `top` Command

`top` is the classic, built-in Unix/Linux utility for **real-time process monitoring**. It provides a dynamic, continuously updating view of system processes and resource usage.

### Launching `top`

```bash
top
```

### Understanding the `top` Header

The top section of the `top` display shows system-wide summary information:

```
top - 14:32:01 up 3 days, 2:15,  2 users,  load average: 0.52, 0.58, 0.59
Tasks: 235 total,   1 running, 234 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  1.0 sy,  0.0 ni, 93.5 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
MiB Mem :  15923.5 total,   2341.2 free,   8234.1 used,   5348.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   6890.3 avail Mem
```

| Section | Description |
|---------|-------------|
| **Uptime** | `up 3 days, 2:15` – Time since last boot |
| **Users** | Number of logged-in users |
| **Load Average** | 1, 5, and 15-minute averages of runnable processes |
| **Tasks** | Total, running, sleeping, stopped, zombie processes |
| **CPU** | Breakdown of CPU time: user, system, nice, idle, wait, etc. |
| **Memory** | Total, free, used, buffer/cache memory in MiB |
| **Swap** | Total, free, used swap space |

#### CPU Time Breakdown

| Abbreviation | Meaning |
|--------------|---------|
| `us` | User CPU time (non-kernel processes) |
| `sy` | System CPU time (kernel processes) |
| `ni` | Nice CPU time (low-priority user processes) |
| `id` | Idle CPU time |
| `wa` | I/O Wait time (waiting for disk/network) |
| `hi` | Hardware interrupts |
| `si` | Software interrupts |
| `st` | Steal time (time stolen by virtual machines) |

### Process List Columns

The lower section lists individual processes:

| Column | Description |
|--------|-------------|
| `PID` | Process ID |
| `USER` | Process owner |
| `PR` | Priority (kernel) |
| `NI` | Nice value (user-adjustable priority) |
| `VIRT` | Virtual memory size (total allocated) |
| `RES` | Resident memory (physical RAM used) |
| `SHR` | Shared memory |
| `S` | Process state (see [Process States](#process-states-explained)) |
| `%CPU` | CPU usage percentage |
| `%MEM` | Memory usage percentage |
| `TIME+` | Total CPU time consumed |
| `COMMAND` | Command that started the process |

### Interactive Commands in `top`

While `top` is running, use these keys:

| Key | Action |
|-----|--------|
| `P` | Sort by **CPU** usage (default) |
| `M` | Sort by **Memory** usage |
| `T` | Sort by **Time** (cumulative CPU time) |
| `N` | Sort by **PID** |
| `k` | **Kill** a process (prompts for PID and signal) |
| `r` | **Renice** a process (change priority) |
| `1` | Toggle **per-CPU** display (shows each core individually) |
| `c` | Toggle **full command path** display |
| `H` | Toggle **thread** display |
| `f` | Select which **fields** to display |
| `h` | Show **help** |
| `q` | **Quit** |

### Batch Mode

For scripting and logging, run `top` in batch mode:

```bash
# Capture a single snapshot
top -bn1 | head -20

# Monitor specific processes
top -bn1 -p 1234,5678

# Log to file every 5 seconds
top -bd5 -n12 -b > top_log.txt
```

---

## The `htop` Command

`htop` is an **interactive, ncurses-based process viewer** that improves upon `top` with a more user-friendly, colorful, and navigable interface. It supports mouse interaction, vertical/horizontal scrolling, and tree views.

### Installing `htop`

`htop` is not always installed by default.

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install htop

# RHEL / CentOS / Fedora
sudo dnf install htop

# Arch Linux
sudo pacman -S htop

# macOS
brew install htop
```

### Understanding the `htop` Interface

The `htop` screen is divided into three main sections:

#### 1. Top-Left: CPU & Memory Bars

```
  1  [|||||||||||||||||||||||||||||||||||||||||   85.2%]
  2  [||||||||||||||||                          35.4%]
  Mem [||||||||||||||||||||||||||||||            12.3G/15.6G]
  Swp [                                          0K/2.00G]
```

- **CPU Bars**: One bar per CPU core. The ticks show load distribution.
- **Memory Bar**: Shows used vs. total RAM.
- **Swap Bar**: Shows used vs. total swap space.

#### 2. Top-Right: Uptime & Load Average

```
Uptime: 3 days, 02:15:33
Load average: 0.52 0.58 0.59
Tasks: 235, 1 running
```

#### 3. Bottom: Process List

| Column | Description |
|--------|-------------|
| `PID` | Process ID |
| `USER` | Process owner |
| `PRI` | Kernel priority |
| `NI` | Nice value |
| `VIRT` | Virtual memory size |
| `RES` | Resident (physical) memory |
| `SHR` | Shared memory |
| `S` | State |
| `CPU%` | CPU usage percentage |
| `MEM%` | Memory usage percentage |
| `TIME+` | CPU time consumed |
| `Command` | Process command |

### Color Coding

#### CPU Bar Colors

| Color | Meaning |
|-------|---------|
| 🟢 **Green** | Normal (user) processes |
| 🔴 **Red** | Kernel processes |
| 🔵 **Blue** | Low-priority (nice) processes |

#### Memory Bar Colors

| Color | Meaning |
|-------|---------|
| 🟢 **Green** | Used memory pages |
| 🔵 **Blue** | Buffer pages |
| 🟡 **Yellow** | Cache pages |

### Interactive Commands in `htop`

| Key | Action |
|-----|--------|
| `F1` / `h` | **Help** screen |
| `F2` / `S` | **Setup** – customize colors, meters, columns |
| `F3` / `/` | **Search** processes by name |
| `F4` / `\` | **Filter** processes (incremental) |
| `F5` / `t` | **Tree** view (parent-child hierarchy) |
| `F6` / `<` / `>` | **Sort** by column |
| `F7` / `]` | Increase priority (lower nice value) – **root only** |
| `F8` / `[` | Decrease priority (higher nice value) |
| `F9` / `k` | **Kill** process (select signal from menu) |
| `F10` / `q` | **Quit** |
| `Space` | **Tag** a process (for bulk operations) |
| `u` | Filter by **user** |
| `M` | Sort by **memory** (top compatibility) |
| `P` | Sort by **CPU** (top compatibility) |
| `T` | Sort by **time** (top compatibility) |
| `I` | **Invert** sort order |
| `K` | Toggle **kernel threads** |
| `H` | Toggle **user threads** |
| `p` | Show full **paths** to executables |
| `Z` | **Pause/resume** process updates |

### Command-Line Options

```bash
# Set update delay (in tenths of a second)
htop -d 10          # 1.0 second delay

# Show only a specific user's processes
htop -u username

# Show only specific PIDs
htop -p 1234,5678

# Sort by column on startup
htop -s PERCENT_CPU

# Tree view
htop -t

# Monochrome mode
htop --no-color
```

---

## Process States Explained

The `S` column in both `top` and `htop` shows the current state of each process:

| State | Name | Description |
|-------|------|-------------|
| `R` | **Running** | Currently executing or waiting for CPU time |
| `S` | **Sleeping** | Interruptible sleep – waiting for an event (most common) |
| `D` | **Uninterruptible Sleep** | Usually waiting for I/O (cannot be killed) |
| `Z` | **Zombie** | Defunct process – terminated but not reaped by parent |
| `T` | **Stopped** | Stopped by job control signal (e.g., `Ctrl+Z`) |
| `t` | **Traced** | Stopped by debugger during tracing |
| `X` | **Dead** | Should never be seen |

> 💡 **Note:** `ps` shows substates like `Ss` (sleeping, session leader) or `R+` (running, foreground process group).

### Zombie Processes

A **zombie process** has finished execution but still has an entry in the process table because its parent hasn't read its exit status via `wait()`. Zombies consume minimal resources but indicate a bug in the parent process. They disappear when the parent terminates.

---

## Memory Metrics: VIRT, RES, SHR

Understanding memory usage is critical for diagnosing performance issues:

| Metric | Full Name | Description |
|--------|-----------|-------------|
| `VIRT` | **Virtual Memory** | Total memory the process can access, including swapped-out memory, shared libraries, and memory-mapped files. This is the largest number and can exceed physical RAM. |
| `RES` | **Resident Memory** | Actual physical RAM the process is using right now. This is the most important metric for real memory consumption. |
| `SHR` | **Shared Memory** | Memory shared with other processes (e.g., shared libraries). Subtracting this from RES gives a rough idea of private memory usage. |

> 💡 **Rule of thumb:** Focus on `RES` when investigating memory hogs. `VIRT` can be misleadingly large.

---

## Load Average Explained

The **load average** appears in both tools and represents the average number of processes that are either:
- Currently running on the CPU, **or**
- Waiting for CPU time (in the run queue)

```
load average: 0.52, 0.58, 0.59
#            1min  5min  15min
```

### Interpreting Load Average

| Scenario | Interpretation |
|----------|----------------|
| **Load < number of cores** | System has spare capacity |
| **Load = number of cores** | System is fully utilized |
| **Load > number of cores** | Processes are waiting – potential slowdown |

**Example:** On a 4-core CPU:
- Load `2.0` → 50% utilized, comfortable
- Load `4.0` → 100% utilized, fully loaded
- Load `8.0` → 200% overloaded, significant queuing

> 💡 **Pro Tip:** Combine load average with CPU `%id` (idle). High load + high idle = processes waiting on I/O (disk/network bottleneck).

---

## `top` vs `htop`: Comparison

| Feature | `top` | `htop` |
|---------|-------|--------|
| **Preinstalled** | ✅ Yes (virtually all Unix/Linux) | ❌ No (requires installation) |
| **Interface** | Text-based, static columns | ncurses-based, scrollable |
| **Mouse Support** | ❌ No | ✅ Yes |
| **Color Coding** | Basic | Rich, customizable |
| **Tree View** | ❌ No | ✅ Yes (F5) |
| **Search/Filter** | Limited | ✅ Incremental search & filter |
| **Kill Process** | Manual PID entry | ✅ Visual signal selection |
| **Bulk Operations** | ❌ No | ✅ Tag multiple processes |
| **Setup/Customization** | Limited | ✅ Extensive (F2) |
| **Scroll Process List** | ❌ No | ✅ Vertical & horizontal |
| **CPU Bars** | ❌ No | ✅ Visual per-core bars |
| **Batch Mode** | ✅ Yes (`-b`) | ❌ No |
| **Best For** | Quick checks, scripting | Interactive exploration |

---

## Practical Workflows

### 🔥 Finding a Runaway Process

```bash
# Step 1: Launch htop
htop

# Step 2: Sort by CPU (F6 → CPU%) or Memory (F6 → MEM%)

# Step 3: Identify the culprit process

# Step 4: Check its user, command, and parent (F5 for tree view)

# Step 5: Decide: investigate further, renice, or kill (F9)
```

### 🧹 Killing a Frozen Application

```bash
# Using top
top
# Press 'k', enter PID, enter signal (default 15 = SIGTERM, 9 = SIGKILL)

# Using htop
htop
# Navigate to process, press F9, select signal, Enter
```

### 📊 Monitoring Specific Processes

```bash
# top: monitor specific PIDs
top -p 1234,5678,9012

# htop: filter by user
htop -u www-data

# htop: show only specific PIDs
htop -p 1234,5678
```

### 📝 Capturing a Snapshot for Analysis

```bash
# Using ps for a detailed snapshot
ps aux --sort=-%cpu | head -20 > process_snapshot.txt

# Using top in batch mode
top -bn1 > top_snapshot.txt

# Using htop (no native batch mode – use ps or top instead)
```

### 🌳 Viewing Process Hierarchy

```bash
# htop tree view
htop -t

# Or use pstree
pstree -a

# Or ps with forest view
ps -ef --forest
```

### 🔍 Searching for a Specific Process

```bash
# Using ps and grep
ps aux | grep firefox

# Using htop interactive search
htop
# Press F3, type 'firefox'
```

---

## Quick Reference Cheat Sheet

### `top` Cheat Sheet

```
P  → Sort by CPU
M  → Sort by Memory
T  → Sort by Time
N  → Sort by PID
k  → Kill process
r  → Renice process
1  → Toggle per-CPU view
c  → Toggle command path
H  → Toggle threads
f  → Select fields
h  → Help
q  → Quit
```

### `htop` Cheat Sheet

```
F1 / h   → Help
F2 / S   → Setup
F3 / /   → Search
F4 / \  → Filter
F5 / t   → Tree view
F6 / <>  → Sort
F7 / ]   → Increase priority (root)
F8 / [   → Decrease priority
F9 / k   → Kill
F10 / q  → Quit

u  → Filter by user
M  → Sort by memory
P  → Sort by CPU
T  → Sort by time
I  → Invert sort
K  → Toggle kernel threads
H  → Toggle user threads
Z  → Pause updates
```

---

## 📚 Additional Resources

- [Linux `top` Manual Page](https://man7.org/linux/man-pages/man1/top.1.html)
- [Linux `htop` Manual Page](https://man7.org/linux/man-pages/man1/htop.1.html)
- [htop Author's Website](https://hisham.hm/htop/)
- [Linux Process Management – Red Hat Documentation](https://access.redhat.com/documentation/)

---

## ✅ Summary

| Tool | Best Use Case |
|------|---------------|
| `ps` | Quick snapshot, scripting, automation |
| `top` | Live monitoring on any system (always available) |
| `htop` | Interactive exploration, visual monitoring, process management |

> 🎯 **Pro Tip:** Start with `ps` for a quick look, use `top` when you need live data on a minimal system, and switch to `htop` when you want a powerful, interactive experience for deep investigation.

---

*Generated with ❤️ for the Linux community. Based on the tutorial video [Linux Processes, top & htop](https://youtu.be/nQhRRLgLFaQ?si=MY1wozAiMJMIEY5p).*
