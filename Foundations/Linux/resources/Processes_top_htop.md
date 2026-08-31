#  Linux Processes, `top` & `htop` — The Complete Guide

> A comprehensive, well-documented reference for understanding Linux processes and mastering the `top` and `htop` process monitoring tools.

---

## 1. Understanding Linux Processes

### What is a Process?

A **process** is an instance of a program that is being executed. When you run a command or start an application, the operating system creates a process to manage that program's execution. Each process has its own:

- **Memory space** (virtual memory)
- **Process ID (PID)** — a unique identifier
- **Parent Process ID (PPID)** — the process that spawned it
- **State** — what the process is currently doing
- **Priority** — how urgently the CPU should handle it
- **Owner** — the user who launched it

Linux is a **multitasking** operating system, meaning multiple processes can run simultaneously. Even on a single-core CPU, this is achieved through **time slicing** — the kernel rapidly switches between processes, giving each a small slice of CPU time (typically a few milliseconds). This happens so fast that it appears as if all processes are running at the same time.

>  **Key Insight:** The Linux kernel internally refers to processes as **tasks**. That's why tools like `htop` use "Tasks" instead of "Processes" in their headers.

---

### Process Identification (PID & PPID)

| Term | Description | Example |
|------|-------------|---------|
| **PID** | Process ID — a unique number assigned to every running process | `PID 1234` |
| **PPID** | Parent Process ID — the PID of the process that created this one | `PPID 1` (init/systemd) |
| **TID** | Thread ID — identifier for individual threads within a process | `TID 1235` |

Every process (except the very first one, `init` or `systemd` with PID `1`) is created by another process. When a process creates a new process, it becomes the **parent**, and the new process is the **child**.

**Example — SSH Session Chain:**

```
systemd (PID 1)
  └── sshd (SSH daemon)
        └── sshd: root@pts/0 (session process)
              └── -bash (login shell)
                    └── htop
```

>  **The dash before `-bash`:** When a shell is launched as `-bash` (with a leading dash), it is a **login shell**. This causes it to read a different set of configuration files (`/etc/profile`, `~/.bash_profile`, etc.) compared to a non-login shell.

---

### Process States

Every process in Linux exists in one of several states. Understanding these is critical for diagnosing system behavior:

| State | Name | Description |
|-------|------|-------------|
| **R** | Running / Runnable | The process is currently executing on the CPU or waiting in the run queue to be scheduled. |
| **S** | Interruptible Sleep | The process is waiting for an event to complete (e.g., I/O, user input). It can be interrupted by signals. This is the most common state. |
| **D** | Uninterruptible Sleep | The process is waiting for I/O and **cannot** be interrupted. Usually indicates disk or network I/O. A process stuck in D-state can be problematic. |
| **Z** | Zombie (Defunct) | The process has terminated, but its entry remains in the process table because its parent hasn't read its exit status yet. |
| **T** | Stopped | The process has been stopped by a job control signal (e.g., `Ctrl+Z`). |
| **t** | Traced / Stopped (Debugger) | The process is being traced by a debugger. |
| **X** | Dead | The process is dead (should never be visible). |

**Substate Modifiers:**

When using `ps`, you may see additional characters after the state letter:

- `s` — session leader (e.g., `Ss`)
- `+` — foreground process group (e.g., `R+`)
- `<` — high priority (not nice)
- `N` — low priority (nice)
- `l` — multi-threaded

**Example:**

```bash
$ ps x
  PID TTY      STAT   TIME COMMAND
 1688 ?        Ss     0:00 /lib/systemd/systemd --user
 1724 ?        S      0:01 sshd: vagrant@pts/0
 1725 pts/0    Ss     0:00 -bash
 2628 pts/0    R+     0:00 ps x
```

---

### Process Hierarchy (Parent & Child)

Processes form a **tree structure** based on parent-child relationships. When a process launches another:

1. The parent calls `fork()` to create a copy of itself.
2. The child then calls `exec()` to load the new program into memory.
3. The parent typically calls `wait()` to pause until the child finishes.

**Viewing the Process Tree:**

```bash
# Using ps with forest view
ps f

# Using pstree
pstree -a

# In htop, press F5 to toggle tree view
```

**Example Output:**

```bash
$ ps f
  PID TTY      STAT   TIME COMMAND
12472 pts/0    Ss     0:00 -bash
12684 pts/0    R+     0:00  \_ ps f
```

>  **Why it matters:** Understanding the tree helps you identify which service spawned a problematic process, or which shell is running a runaway script.

---

### Process Memory Model

Each process has its own virtual memory space, divided into several segments:

| Memory Type | Description |
|-------------|-------------|
| **VIRT** (Virtual Memory) | The total amount of virtual memory allocated to the process. This includes memory swapped out, memory mapped from files, and shared libraries. |
| **RES** (Resident Memory) | The actual physical RAM the process is using. This is the most important metric for memory consumption. |
| **SHR** (Shared Memory) | Memory shared with other processes (e.g., shared libraries, shared memory segments). |
| **CODE** (Text) | The executable machine code of the program. |
| **DATA** | Initialized and uninitialized global/static variables. |
| **STACK** | Local variables, function parameters, and return addresses. |
| **HEAP** | Dynamically allocated memory (via `malloc`, `new`, etc.). |

>  **Key Insight:** A process can have high VIRT but low RES if it has mapped large files or allocated memory it hasn't actually touched yet. Focus on **RES** for real memory usage.

---

### Signals & Inter-Process Communication

Signals are a form of **inter-process communication (IPC)** used to notify processes of events. They are essentially numeric messages with names.

| Signal | Number | Name | Description |
|--------|--------|------|-------------|
| **SIGHUP** | 1 | HUP | Hang up — often used to tell daemons to reload configuration. |
| **SIGINT** | 2 | INT | Interrupt — sent when you press `Ctrl+C`. |
| **SIGKILL** | 9 | KILL | Kill — forcefully terminates a process. **Cannot be caught or ignored.** |
| **SIGTERM** | 15 | TERM | Terminate — polite request to shut down. The default signal for `kill`. |
| **SIGSTOP** | 19 | STOP | Stop — pauses a process. Cannot be caught. |
| **SIGCONT** | 18 | CONT | Continue — resumes a stopped process. |

**Sending Signals:**

```bash
# Default (SIGTERM)
kill 1234

# Specific signal
kill -9 1234        # SIGKILL
kill -SIGTERM 1234  # SIGTERM
kill -HUP 1234      # SIGHUP (reload config)

# In htop: press F9, select signal, then Enter
```

>  **Best Practice:** Always try `SIGTERM` (15) first. Only use `SIGKILL` (9) when a process refuses to die, because SIGKILL doesn't allow the process to clean up resources.

---

## 2. The `top` Command

### What is `top`?

`top` is the classic, built-in **real-time process monitoring** tool for Linux. It provides a continuously updating view of system processes, CPU usage, memory usage, and load averages. Unlike `ps` (which gives a static snapshot), `top` refreshes dynamically, making it ideal for live troubleshooting.

**Launch `top`:**

```bash
top
```

---

### Understanding the `top` Display

```
top - 10:35:22 up 2 days,  4:12,  2 users,  load average: 0.23, 0.18, 0.15
Tasks: 234 total,   1 running, 233 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.8 sy,  0.0 ni, 96.7 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7854.8 total,   1234.5 free,   3456.7 used,   3163.6 buff/cache
MiB Swap:   2048.0 total,   2047.5 free,      0.5 used.   3876.4 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1234 www-data  20   0  425680  52340  12340 S  45.2  0.7   2:34.56 nginx
 5678 root      20   0  215000  31200   8900 S  12.1  0.4   0:45.21 php-fpm
 ...
```

#### Header Breakdown

**Line 1 — System Summary:**

| Field | Meaning |
|-------|---------|
| `10:35:22` | Current system time |
| `up 2 days, 4:12` | System uptime (how long since last boot) |
| `2 users` | Number of logged-in users |
| `load average: 0.23, 0.18, 0.15` | Average number of runnable processes over the last 1, 5, and 15 minutes |

>  **Load Average Explained:** The load average represents the average number of processes that are either running or waiting to run. On a single-core CPU, a load of `1.0` means 100% utilization. On a 4-core CPU, `4.0` is full capacity. Values above your core count indicate the system is overloaded.

**Line 2 — Task Summary:**

| Field | Meaning |
|-------|---------|
| `234 total` | Total processes |
| `1 running` | Processes in R-state |
| `233 sleeping` | Processes in S-state |
| `0 stopped` | Processes in T-state |
| `0 zombie` | Zombie processes |

**Line 3 — CPU Usage:**

| Field | Meaning |
|-------|---------|
| `us` | User space CPU time (normal processes) |
| `sy` | System (kernel) CPU time |
| `ni` | Nice (low-priority user processes) |
| `id` | Idle CPU time |
| `wa` | Waiting for I/O |
| `hi` | Hardware interrupts |
| `si` | Software interrupts |
| `st` | Steal time (time stolen by virtual machines/hypervisor) |

**Lines 4–5 — Memory & Swap:**

| Field | Meaning |
|-------|---------|
| `total` | Total installed memory |
| `free` | Completely unused memory |
| `used` | Memory currently in use by processes |
| `buff/cache` | Memory used for buffers and cache (can be reclaimed) |
| `avail Mem` | Memory available for starting new processes without swapping |
| `Swap` | Disk space used when RAM is full |

#### Process Columns

| Column | Description |
|--------|-------------|
| `PID` | Process ID |
| `USER` | Process owner |
| `PR` | Priority (kernel scheduling priority) |
| `NI` | Nice value (user-adjustable priority modifier) |
| `VIRT` | Virtual memory size |
| `RES` | Resident (physical) memory size |
| `SHR` | Shared memory size |
| `S` | Process state (R, S, D, Z, T) |
| `%CPU` | Percentage of CPU time used |
| `%MEM` | Percentage of physical RAM used |
| `TIME+` | Total CPU time consumed since process started |
| `COMMAND` | Command that started the process |

---

### Interactive Commands in `top`

While `top` is running, press these keys:

| Key | Action |
|-----|--------|
| `q` | Quit `top` |
| `h` or `?` | Show help |
| `Space` | Immediately refresh the display |
| `P` | Sort by CPU usage (default) |
| `M` | Sort by memory usage |
| `T` | Sort by total CPU time |
| `N` | Sort by PID (numerical) |
| `k` | Kill a process (prompts for PID and signal) |
| `r` | Renice a process (change its priority) |
| `u` | Filter by specific user |
| `1` | Toggle per-CPU/core display |
| `c` | Toggle between command name and full path |
| `d` or `s` | Change refresh delay (in seconds) |
| `f` | Select which columns to display |
| `W` | Write current configuration to `~/.toprc` |
| `b` | Toggle bold highlighting |
| `x` | Highlight the sorted column |
| `y` | Highlight running tasks |

---

### Batch Mode for Scripting

`top` can run in non-interactive **batch mode**, making it useful for logging and scripting:

```bash
# Capture one snapshot and save to file
top -b -n 1 > /tmp/top-snapshot.txt

# Skip header, get only process lines
top -b -n 1 | tail -n +8 > /tmp/processes.txt

# Get top 10 CPU consumers
top -b -n 1 -o %CPU | head -17 | tail -10

# Monitor a specific user's processes
top -b -n 1 -u www-data

# Run with 2-second delay, 5 iterations
top -b -d 2 -n 5 > /tmp/top-log.txt
```

>  **Tip:** Use `top -b` in cron jobs or monitoring scripts to capture periodic system snapshots.

---

## 3. The `htop` Command

### What is `htop`?

`htop` is an **enhanced, interactive process viewer** that serves as a more user-friendly alternative to `top`. Built with `ncurses`, it provides:

- Color-coded visual meters for CPU, memory, and swap
- Scrollable process list (vertical and horizontal)
- Mouse support
- Tree view of process hierarchy
- Ability to select multiple processes and act on them simultaneously
- No need to type PIDs for killing or renicing

>  **Key Advantage:** `htop` is fully interactive. You can navigate with arrow keys, click with your mouse, and manage processes without memorizing cryptic single-letter commands.

---

### Installing `htop`

`htop` is not always installed by default. Here's how to install it on major distributions:

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install htop

# RHEL / CentOS / Fedora
sudo dnf install htop
# or for older versions:
sudo yum install htop

# Arch Linux
sudo pacman -S htop

# macOS (via Homebrew)
brew install htop
```

**Launch:**

```bash
htop
```

---

### Understanding the `htop` Display

```
  1  [||||||||||||||||||||||||||||||||||||||||||||||||||   85.2%]   2  [|||||||||||||||||||||||||||||||||                      52.1%]
  3  [||||||||||||||||||||                                   31.4%]   4  [|||||||||||||||||||||||||||||||||||||||||||||||||||||  92.3%]
Mem [||||||||||||||||||||||||||||||||||||||||||             4.12G/7.85G]
Swp [                                                      0.00K/2.00G]

  PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
 1234 www-data   20   0  425M  52.3M 12.3M R 45.2  0.7  2:34.56 nginx: worker process
 5678 root       20   0  215M  31.2M  8.9M S 12.1  0.4  0:45.21 php-fpm: pool www
 ...

F1Help  F2Setup F3SearchF4FilterF5Tree  F6SortByF7Nice-F8Nice+F9Kill  F10Quit
```

#### Header Meters

**CPU Bars:**

Each bar represents one CPU core. The colored segments indicate:

| Color | Meaning |
|-------|---------|
|  Green | Normal (user) processes |
|  Red | Kernel/system processes |
|  Blue | Low-priority (nice) processes |
|  Yellow / Orange | IRQ time (hardware interrupts) |
|  Gray | I/O wait |

**Memory (Mem) Bar:**

| Color | Meaning |
|-------|---------|
|  Green | Used memory pages |
|  Blue | Buffer pages |
|  Yellow / Orange | Cache pages |

**Swap (Swp) Bar:**

Shows swap space usage. If this is consistently high, your system is under memory pressure.

#### Process Columns

| Column | Description |
|--------|-------------|
| `PID` | Process ID |
| `USER` | Process owner |
| `PRI` | Kernel scheduling priority |
| `NI` | Nice value (user-adjustable) |
| `VIRT` | Virtual memory size |
| `RES` | Resident (physical) memory |
| `SHR` | Shared memory |
| `S` | Process state |
| `CPU%` | CPU usage percentage |
| `MEM%` | Memory usage percentage |
| `TIME+` | Total CPU time consumed |
| `Command` | Full command line |

---

### Interactive Commands in `htop`

| Key | Action |
|-----|--------|
| `F1` / `h` | Help screen |
| `F2` / `S` | Setup — configure meters, colors, columns |
| `F3` / `/` | Search for a process by name |
| `F4` / `\` | Filter processes (show only matching) |
| `F5` / `t` | Toggle tree view (process hierarchy) |
| `F6` / `<` / `>` | Sort by column |
| `F7` / `]` | Decrease nice value (increase priority) — requires root for values below 0 |
| `F8` / `[` | Increase nice value (decrease priority) |
| `F9` / `k` | Kill process — select signal from menu |
| `F10` / `q` | Quit |
| `Space` | Tag/untag a process (for bulk operations) |
| `c` | Tag current process and all its children |
| `U` | Untag all processes |
| `u` | Show only processes for a specific user |
| `I` | Invert sort order |
| `H` | Toggle display of user threads |
| `K` | Toggle display of kernel threads |
| `p` | Show full paths to programs |
| `Z` | Pause/resume process updates |
| `s` | Trace system calls with `strace` (if installed) |
| `l` | Show open files with `lsof` (if installed) |
| `a` | Set CPU affinity (which cores a process can use) |
| `N` | Sort by PID |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `T` | Sort by time |
| `F` | Follow process (keep it visible as it moves) |
| `+` / `-` | Expand/collapse tree branches |
| `*` | Expand/collapse all tree children |

---

### Tree View

Press `F5` to toggle the **tree view**, which shows the parent-child relationships between processes:

```
init (PID 1)
├── systemd-journal
├── sshd
│   └── sshd: root@pts/0
│       └── -bash
│           └── htop
├── nginx
│   ├── nginx: master process
│   ├── nginx: worker process
│   └── nginx: worker process
└── php-fpm
    ├── php-fpm: master process
    └── php-fpm: pool www
```

>  **Tree View Benefits:**
> - See which service spawned a process
> - Identify runaway child processes
> - Understand application architecture at a glance

**Toggle Threads:**

- `Shift + H` — Toggle user threads
- `Shift + K` — Toggle kernel threads

By default, `htop` shows threads, which can make the list look crowded. Press `Shift + H` to hide them and see only processes.

---

### Configuration & Customization

Press `F2` (Setup) to customize `htop`:

**Meters (F2 → Meters):**
- Add/remove CPU, memory, disk, network meters
- Choose between bar, graph, LED, or text styles

**Display Options (F2 → Display options):**
- Highlight program basename
- Highlight threads
- Show program path
- Shadow other users' processes

**Colors (F2 → Colors):**
- Choose from multiple color schemes (Default, Monochromatic, Black Night, etc.)

**Columns (F2 → Columns):**
- Add, remove, or reorder columns
- Available columns include: PID, USER, PRI, NI, VIRT, RES, SHR, S, CPU%, MEM%, TIME+, Command, and many more

**Configuration File:**

`htop` saves settings to:

```
~/.config/htop/htoprc
```

You can edit this file directly or use the `F2` menu.

---

## 4. `top` vs `htop` — Comparison

| Feature | `top` | `htop` |
|---------|-------|--------|
| **Availability** | Pre-installed on virtually all Linux systems | Must be installed separately |
| **Interface** | Text-based, minimal | Colorful, ncurses-based GUI |
| **Navigation** | Keyboard shortcuts only | Arrow keys + mouse support |
| **Scrolling** | Cannot scroll process list | Full vertical and horizontal scrolling |
| **Tree View** | Not available | Built-in (F5) |
| **Search/Filter** | Limited | Full search (F3) and filter (F4) |
| **Kill Process** | Type `k`, then PID | Select process, press F9 |
| **Renice** | Type `r`, then PID | Select process, press F7/F8 |
| **Multiple Select** | No | Yes (Space to tag) |
| **Configuration** | Via `f` key and `~/.toprc` | Rich setup menu (F2) and `~/.config/htop/htoprc` |
| **Batch Mode** | Yes (`-b` flag) | Limited |
| **Resource Usage** | Very lightweight | Slightly heavier (ncurses overhead) |
| **Best For** | Quick checks, scripting, minimal systems | Interactive exploration, visual monitoring, learning |

>  **When to use which:**
> - Use **`ps`** for one-time snapshots and scripting
> - Use **`top`** when you need a lightweight, always-available monitor or batch output
> - Use **`htop`** when you want an interactive, visual, and user-friendly experience

---

## 5. Practical Workflows & Use Cases

### Finding Runaway Processes

**Scenario:** Your server is slow, and you need to find what's consuming resources.

```bash
# Step 1: Open htop
htop

# Step 2: Sort by CPU (F6 → CPU%) or press 'P'
# Step 3: Identify the process with highest CPU%
# Step 4: Note the PID and Command
# Step 5: Investigate further with: lsof -p <PID> or strace -p <PID>
```

**With `top`:**

```bash
top
# Press 'P' to sort by CPU
# Press 'c' to see full command path
```

---

### Killing Unresponsive Processes

**Scenario:** A process is frozen and not responding.

**In `htop`:**

1. Select the process with arrow keys
2. Press `F9` (Kill)
3. Select `SIGTERM` (15) first — polite termination
4. If it persists, repeat with `SIGKILL` (9) — forceful termination

**In `top`:**

1. Press `k`
2. Enter the PID
3. Enter signal number (default is 15)

**Command line:**

```bash
# Polite termination
kill 1234

# Force kill (use as last resort)
kill -9 1234

# Kill by name
killall nginx
pkill -f "python app.py"
```

---

### Monitoring Memory Pressure

**Scenario:** System is running out of memory.

```bash
# In htop: Sort by memory (F6 → PERCENT_MEM) or press 'M'
# Watch the Mem bar at the top
# Check if Swap is being used (indicates RAM exhaustion)

# In top: Press 'M' to sort by memory
```

**Key indicators of memory pressure:**

- High `used` memory with low `free` and `avail Mem`
- Swap usage above 0%
- Processes with high `%MEM` values
- High `buff/cache` is normal — Linux uses free RAM for cache

---

### Checking Process Hierarchy

**Scenario:** You need to understand which process spawned another.

```bash
# Method 1: htop tree view
htop
# Press F5

# Method 2: ps forest
ps f

# Method 3: pstree
pstree -p -a | grep nginx

# Method 4: Find parent of specific PID
ps -o pid,ppid,cmd -p 1234
cat /proc/1234/status | grep PPid
```

---

### Monitoring a Specific Service

```bash
# In top: Filter by user
# Press 'u', then type username (e.g., 'www-data')

# In htop: Filter by user
# Press 'u', select user from list

# Command line: Monitor specific process
watch -n 1 'ps -p $(pgrep nginx | head -1) -o pid,ppid,cmd,%cpu,%mem'
```

---

### Capturing Snapshots for Documentation

```bash
# Capture full process list
ps aux > /tmp/process-snapshot-$(date +%Y%m%d-%H%M%S).txt

# Capture top output for a ticket
top -b -n 1 > /tmp/top-report.txt

# Capture htop-like info without htop
ps aux --sort=-%cpu | head -20

# Periodic logging
while true; do
    echo "=== $(date) ===" >> /tmp/process-log.txt
    ps aux --sort=-%cpu | head -5 >> /tmp/process-log.txt
    sleep 60
done
```

---

## 6. Glossary of Key Terms

| Term | Definition |
|------|------------|
| **Process** | An instance of a running program |
| **PID** | Process ID — unique identifier for a process |
| **PPID** | Parent Process ID — the process that created this one |
| **Thread** | A lightweight unit of execution within a process |
| **Nice Value** | A value (-20 to 19) that adjusts process priority. Lower = higher priority. |
| **Priority (PRI)** | The kernel's internal scheduling priority |
| **Load Average** | Average number of runnable processes over 1, 5, and 15 minutes |
| **Virtual Memory (VIRT)** | Total address space allocated to a process |
| **Resident Memory (RES)** | Actual physical RAM used by a process |
| **Shared Memory (SHR)** | Memory shared between multiple processes |
| **Swap** | Disk space used as overflow when RAM is full |
| **Zombie Process** | A terminated process whose parent hasn't read its exit status |
| **Orphan Process** | A process whose parent has terminated (reparented to init) |
| **Daemon** | A background process that runs continuously (e.g., `sshd`, `nginx`) |
| **Signal** | A software interrupt sent to a process to notify it of an event |
| **Fork** | System call to create a new process by duplicating the current one |
| **Exec** | System call to replace the current process image with a new one |
| **Time Slice** | The amount of CPU time allocated to a process before switching |
| **CPU Affinity** | Binding a process to specific CPU cores |
| **I/O Wait** | Time the CPU spends waiting for disk or network I/O |

---

## 7. Quick Reference Cheat Sheet

### `top` Cheat Sheet

```bash
# Launch
top

# Common interactive commands (while top is running)
P          # Sort by CPU
M          # Sort by memory
T          # Sort by time
N          # Sort by PID
k          # Kill process (enter PID, then signal)
r          # Renice process (enter PID, then nice value)
u          # Filter by user
1          # Toggle per-CPU view
c          # Toggle command/full path
d          # Change refresh delay
q          # Quit

# Batch mode
top -b -n 1                    # One snapshot
top -b -n 1 -u www-data        # Snapshot for specific user
top -b -d 2 -n 5 > log.txt     # 5 snapshots, 2s apart
```

### `htop` Cheat Sheet

```bash
# Launch
htop

# Common interactive commands
F1 / h     # Help
F2 / S     # Setup
F3 / /     # Search
F4 / \    # Filter
F5 / t     # Tree view
F6 / < >   # Sort by column
F7 / ]     # Increase priority (lower nice)
F8 / [     # Decrease priority (higher nice)
F9 / k     # Kill process
F10 / q    # Quit
Space      # Tag process (bulk ops)
u          # Filter by user
H          # Toggle threads
K          # Toggle kernel threads
I          # Invert sort
Z          # Pause updates

# Command-line options
htop -u www-data               # Show only www-data processes
htop -p 1234,5678              # Show only specific PIDs
htop -t                        # Start in tree view
htop -d 10                     # 1-second delay (value in tenths)
```

### Process Management Commands

```bash
# List processes
ps aux                         # All processes, BSD format
ps -ef                         # All processes, standard format
ps f                           # Forest view (tree)
ps -u username                 # Processes for specific user

# Find processes
pgrep nginx                    # Get PID by name
pgrep -a nginx                 # Get PID + command line
pgrep -u root nginx            # Match user and name

# Kill processes
kill 1234                      # SIGTERM (default)
kill -9 1234                   # SIGKILL (force)
kill -HUP 1234                 # SIGHUP (reload)
killall nginx                  # Kill by name
pkill -f "python app.py"       # Kill by pattern

# Change priority
nice -n 10 command             # Start with nice value 10
renice -n 5 -p 1234            # Change nice value of running process

# Process info
cat /proc/1234/status          # Detailed process info
cat /proc/1234/cmdline         # Command line arguments
ls -la /proc/1234/fd           # Open file descriptors
```

---

##  Additional Resources

- [Linux `top` Manual Page](https://man7.org/linux/man-pages/man1/top.1.html)
- [Linux `htop` Manual Page](https://man7.org/linux/man-pages/man1/htop.1.html)
- [Linux Process Management — The Linux Documentation Project](https://tldp.org/LDP/tlk/kernel/processes.html)
- [Understanding Linux Load Average](https://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html)
- [YouTube Reference Tutorial](https://youtu.be/nQhRRLgLFaQ)

---

##  License

This guide is provided as educational material. Feel free to share, modify, and use it in your own projects.

---

> **Contributions Welcome!** If you find errors or want to add more content, feel free to open a PR.
>
> *Happy monitoring!* 
