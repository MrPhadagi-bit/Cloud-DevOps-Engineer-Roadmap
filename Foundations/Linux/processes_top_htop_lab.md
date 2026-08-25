#  GitHub Lab: Linux Process Monitoring with `top` & `htop`

> **Lab Objective:** Learn to monitor, inspect, and manage running processes on a Linux system using the built-in `top` command and the enhanced `htop` viewer. By the end of this lab, you will be able to read system resource usage, identify resource-heavy processes, sort and filter process lists, and safely terminate or reprioritize tasks.

---

## 1. Prerequisites

Before starting this lab, ensure you have:

- [ ] Access to a Linux terminal (local VM, WSL, cloud instance, or physical machine)
- [ ] Basic familiarity with the command line (`cd`, `ls`, `sudo`)
- [ ] `htop` installed (see Step 4.1 if missing)

### Check Your Environment

```bash
# Verify you are on Linux
uname -a

# Check your distribution
cat /etc/os-release | head -n 5

# Verify htop is installed (if not, see Section 4.1)
which htop
```

---

## 2. What is a Process?

A **process** is an instance of a running program. Every time you launch an application, run a script, or start a system service, the Linux kernel creates a process to manage that task.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **PID** | Process ID — a unique number assigned to every running process |
| **PPID** | Parent Process ID — the PID of the process that started this one |
| **CPU%** | Percentage of CPU time the process is currently consuming |
| **MEM%** | Percentage of total RAM the process is using |
| **VIRT** | Virtual memory — total memory the process has requested |
| **RES** | Resident memory — actual RAM the process occupies |
| **SHR** | Shared memory — memory shared with other processes |
| **NI (Nice)** | Priority value (-20 highest to +19 lowest) |
| **STAT / S** | Process state: `R`unning, `S`leeping, `Z`ombie, `T`stopped |

### Process States Explained

| State | Meaning |
|-------|---------|
| `R` | **Running** — actively executing on the CPU |
| `S` | **Sleeping** — waiting for an event (interruptible) |
| `D` | **Disk Sleep** — uninterruptible sleep (usually I/O wait) |
| `T` | **Stopped** — paused by a signal (e.g., Ctrl+Z) |
| `Z` | **Zombie** — terminated but not yet reaped by parent |

> 💡 **Think of it this way:** The kernel is the manager, processes are the workers, and `top`/`htop` are the dashboards showing you who is doing what and how hard they are working.

---

## 3. Lab Part 1: The `top` Command

`top` is the classic, built-in process monitor available on virtually every Linux system. It provides a real-time, dynamically updating view of system processes and resource usage.

### 3.1 Launching `top`

```bash
# Start top with default settings
top
```

You will see a screen divided into two sections:
- **Summary Area (top half):** System-wide stats (uptime, load, CPU, memory)
- **Task Area (bottom half):** The list of running processes

> 📝 **Note:** `top` runs interactively. It will keep updating until you quit.

---

### 3.2 Reading the `top` Header (Summary Area)

```
top - 14:32:10 up 3 days,  2:15,  1 user,  load average: 0.45, 0.38, 0.31
Tasks: 142 total,   1 running, 141 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  1.1 sy,  0.0 ni, 96.2 id,  0.3 wa,  0.0 hi,  0.1 si,  0.0 st
MiB Mem :   7822.4 total,   1234.5 free,   3456.7 used,   3131.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   3890.2 avail Mem
```

#### Line-by-Line Breakdown

| Line | Field | Meaning |
|------|-------|---------|
| **Line 1** | `14:32:10` | Current system time |
| | `up 3 days, 2:15` | System uptime |
| | `1 user` | Number of logged-in users |
| | `load average: 0.45, 0.38, 0.31` | Average system load over 1, 5, and 15 minutes |
| **Line 2** | `Tasks: 142 total` | Total processes |
| | `1 running` | Processes actively using CPU |
| | `141 sleeping` | Processes waiting for events |
| | `0 stopped` | Paused processes |
| | `0 zombie` | Dead processes not yet cleaned up |
| **Line 3** | `%Cpu(s)` | CPU usage breakdown |
| | `us` | User processes |
| | `sy` | System/kernel processes |
| | `ni` | Nice (low-priority) processes |
| | `id` | Idle CPU |
| | `wa` | Waiting for I/O |
| | `hi` | Hardware interrupts |
| | `si` | Software interrupts |
| | `st` | Time stolen by virtual machines |
| **Line 4-5** | `MiB Mem` / `MiB Swap` | Memory and swap usage |

> 🔍 **Load Average Tip:** A load average of `1.00` on a single-core system means the CPU is fully utilized. On a 4-core system, `4.00` would mean full utilization. Values below your core count are generally healthy.

---

### 3.3 Reading the Process List (Task Area)

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1234 root      20   0  512000  18432   5328 S   2.3  0.2   0:04.21 sshd
 2871 john      20   0 1256000 215040  43008 S   1.1  2.7   0:31.08 python3
  891 www-data  20   0  421888  56320  12288 S   0.5  0.7   0:12.44 nginx
```

| Column | Description |
|--------|-------------|
| `PID` | Process ID |
| `USER` | Owner of the process |
| `PR` | Priority (lower = higher priority) |
| `NI` | Nice value (-20 to +19) |
| `VIRT` | Virtual memory size (KB) |
| `RES` | Resident (physical RAM) size (KB) |
| `SHR` | Shared memory size (KB) |
| `S` | State (R/S/D/T/Z) |
| `%CPU` | CPU usage percentage |
| `%MEM` | Memory usage percentage |
| `TIME+` | Total CPU time consumed |
| `COMMAND` | Command that started the process |

---

### 3.4 Interactive Commands Inside `top`

While `top` is running, press these keys (no Enter needed):

| Key | Action | Description |
|-----|--------|-------------|
| `q` | **Quit** | Exit top and return to shell |
| `h` or `?` | **Help** | Show interactive commands help |
| `1` | **Toggle CPUs** | Show individual CPU cores vs aggregate |
| `M` | **Sort by Memory** | Sort process list by %MEM |
| `P` | **Sort by CPU** | Sort process list by %CPU (default) |
| `T` | **Sort by Time** | Sort by cumulative CPU time |
| `c` | **Toggle Command** | Show full command path vs just name |
| `z` | **Toggle Color** | Enable/disable color coding |
| `x` | **Highlight Column** | Highlight the current sort column |
| `b` | **Bold** | Toggle bold highlighting |
| `k` | **Kill** | Prompt for PID to kill, then signal |
| `r` | **Renice** | Change priority (nice value) of a process |
| `u` | **Filter User** | Show only processes for a specific user |
| `o` | **Filter** | Add a custom filter condition |
| `V` | **Tree View** | Show processes in parent-child hierarchy |
| `H` | **Toggle Threads** | Show/hide individual threads |
| `L` | **Locate** | Search for a string in the process list |
| `d` or `s` | **Set Delay** | Change refresh interval (in seconds) |
| `W` | **Write Config** | Save current settings to `~/.toprc` |

#### 🧪 Hands-On Exercise: Navigate `top`

1. Launch `top`
2. Press `1` to see individual CPU cores
3. Press `M` to sort by memory usage
4. Press `x` to highlight the sort column
5. Press `z` to enable colors
6. Press `c` to see full command paths
7. Press `V` to see the process tree
8. Press `q` to quit

---

### 3.5 Killing a Process with `top`

```bash
# Inside top, press:
k

# You will be prompted:
PID to signal/kill [default pid = 1234] 

# Type the PID and press Enter, then choose signal:
# 15 = SIGTERM (graceful, recommended first)
# 9  = SIGKILL (force, use only if SIGTERM fails)
```

> ⚠️ **Warning:** Always try `15 (SIGTERM)` first. `9 (SIGKILL)` terminates immediately without cleanup and may cause data loss or file corruption.

---

### 3.6 Renicing a Process with `top`

```bash
# Inside top, press:
r

# You will be prompted:
PID to renice [default pid = 1234]

# Enter the PID, then:
Renice PID 1234 to value: 

# Enter a value from -20 (highest priority) to +19 (lowest)
# Regular users can only increase nice value (lower priority)
# Root can decrease nice value (higher priority)
```

---

### 3.7 Running `top` in Batch Mode (for Scripts)

```bash
# Single snapshot, no interactivity — great for logging
 top -bn1 | head -20

# Monitor specific PIDs
 top -bn1 -p 1234,5678

# Save to file
 top -bn1 > system_snapshot.txt
```

| Flag | Meaning |
|------|---------|
| `-b` | Batch mode (non-interactive) |
| `-n 1` | Update only once |
| `-p` | Monitor specific PIDs |

---

## 4. Lab Part 2: The `htop` Command

`htop` is an enhanced, interactive process viewer. It offers a color-coded, scrollable interface with mouse support, making process management more intuitive than `top`.

### 4.1 Installing `htop`

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install htop -y

# RHEL / CentOS / Fedora
sudo dnf install htop -y

# Arch Linux
sudo pacman -S htop

# macOS (via Homebrew)
brew install htop
```

---

### 4.2 Launching `htop`

```bash
# Standard launch
htop

# Monitor specific user
htop -u username

# Monitor specific PID
htop -p 1234

# Launch in tree view
htop -t

# Set refresh delay (2.0 seconds = 20 tenths)
htop -d 20

# Run as root for full visibility
sudo htop
```

---

### 4.3 Understanding the `htop` Interface

```
  1  [|||||||| 12.5%]   Tasks: 142, 421 thr; 2 running
  2  [|||||||||| 18.3%]  Load average: 0.45 0.38 0.31
  Mem[|||||||||||||||||||||||| 2.34G/7.68G]   Uptime: 03:14:22
  Swp[                        0K/2.00G]

  PID USER     PRI  NI  VIRT   RES   SHR S  CPU% MEM%  TIME+  Command
 1234 root      20   0  512M    18M  5.2M S   2.3  0.2  0:04.21 /usr/sbin/sshd
 2871 john      20   0  1.2G   210M   42M S   1.1  2.7  0:31.08 /usr/bin/python3
  891 www-data  20   0  412M    55M   12M S   0.5  0.7  0:12.44 nginx: worker

  F1Help  F2Setup F3SearchF4FilterF5Tree  F6SortByF7Nice- F8Nice+ F9Kill  F10Quit
```

#### Top Meters (Summary Area)

| Meter | Description |
|-------|-------------|
| **CPU Bars** | One bar per core showing usage. Colors indicate: **green** = user processes, **red** = kernel/system, **blue** = low-priority, **yellow** = IRQ |
| **Memory Bar** | RAM usage: green = used, blue = buffers, yellow = cache |
| **Swap Bar** | Swap space usage |
| **Load Average** | 1, 5, and 15-minute averages |
| **Uptime** | How long the system has been running |
| **Tasks** | Total processes, threads, and running count |

#### Process Columns

| Column | Description |
|--------|-------------|
| `PID` | Process ID |
| `USER` | Process owner |
| `PRI` | Priority |
| `NI` | Nice value |
| `VIRT` | Virtual memory |
| `RES` | Resident (physical) memory |
| `SHR` | Shared memory |
| `S` | State |
| `CPU%` | CPU percentage |
| `MEM%` | Memory percentage |
| `TIME+` | CPU time used |
| `Command` | Process name / command |

---

### 4.4 Interactive Commands Inside `htop`

| Key | Action | Description |
|-----|--------|-------------|
| `F1` or `h` | **Help** | Show help screen with all shortcuts |
| `F2` or `S` | **Setup** | Customize meters, colors, columns |
| `F3` or `/` | **Search** | Search for process by name |
| `F4` or `\` | **Filter** | Live filter — only show matching processes |
| `F5` or `t` | **Tree** | Toggle tree view (parent-child hierarchy) |
| `F6` or `<`/`>` | **Sort By** | Choose sort column |
| `F7` or `]` | **Nice -** | Increase priority (lower nice value) — root only |
| `F8` or `[` | **Nice +** | Decrease priority (raise nice value) |
| `F9` or `k` | **Kill** | Send signal to selected process(es) |
| `F10` or `q` | **Quit** | Exit htop |
| `Space` | **Tag** | Tag/untag a process for bulk actions |
| `c` | **Tag Children** | Tag process and all its children |
| `U` | **Untag All** | Remove all tags |
| `u` | **User Filter** | Show only processes for selected user |
| `p` | **Full Path** | Toggle full path display |
| `s` | **Strace** | Attach strace to selected process |
| `l` | **Lsof** | Show open files for selected process |
| `I` | **Invert Sort** | Reverse the sort order |
| `K` | **Hide Kernel** | Toggle display of kernel threads |
| `H` | **Hide User Threads** | Toggle display of user threads |
| `F` | **Follow** | Keep selection on a process as it moves |
| `Z` | **Pause** | Pause/resume screen updates |
| `PgUp`/`PgDn` | **Scroll** | Scroll through process list |
| `Arrow Keys` | **Navigate** | Move selection up/down/left/right |

> 🖱️ **Mouse Support:** You can also click column headers to sort, click processes to select them, and click the bottom menu items to trigger actions.

---

### 4.5 Step-by-Step: Searching & Filtering

#### Search for a Process (F3)

```bash
# 1. Launch htop
htop

# 2. Press F3 (or /)
# 3. Type the process name, e.g., "nginx"
#    Matches will be highlighted as you type
# 4. Press F3 again to jump to next match
# 5. Press Esc to close search
```

#### Filter by User (u)

```bash
# 1. Inside htop, press 'u'
# 2. A user list appears — select with arrow keys, press Enter
# 3. Only that user's processes are shown
# 4. Press 'u' again and select "All users" to reset
```

#### Live Filter (F4)

```bash
# 1. Inside htop, press F4 (or \)
# 2. Type a filter string, e.g., "mysql"
# 3. Only processes containing "mysql" remain visible
# 4. Press Esc to clear the filter
```

---

### 4.6 Step-by-Step: Killing a Process

```bash
# 1. Launch htop
htop

# 2. Use arrow keys to select the process you want to kill
# 3. Press F9 (or 'k')
#    A signal menu appears on the left
#
#    Common signals:
#    1  SIGINT   - Interrupt (Ctrl+C)
#    9  SIGKILL  - Force kill (no cleanup)
#    15 SIGTERM  - Graceful termination (default, recommended)
#    18 SIGCONT  - Continue (resume) a stopped process
#    19 SIGSTOP  - Stop (pause) a process
#
# 4. Select signal 15 (SIGTERM) first
# 5. Press Enter to confirm
#
# 6. If the process doesn't die, repeat with signal 9 (SIGKILL)
```

> ⚠️ **Best Practice:** Always send `SIGTERM (15)` first. Give the process a few seconds to shut down gracefully. Only escalate to `SIGKILL (9)` if it becomes unresponsive.

---

### 4.7 Step-by-Step: Changing Priority (Renice)

```bash
# 1. Inside htop, select the process with arrow keys
# 2. Press F7 to increase priority (lower nice value)
#    OR press F8 to decrease priority (raise nice value)
#
# 3. You will see the NI column change
#    -20 = highest priority (root only)
#      0 = default priority
#    +19 = lowest priority
#
# Note: Regular users can only RAISE nice values (make processes nicer/slower)
#       Only root can LOWER nice values (make processes more aggressive)
```

---

### 4.8 Tree View (F5)

Tree view shows the parent-child relationship between processes, making it easy to see which daemon spawned which workers.

```bash
# 1. Inside htop, press F5 (or 't')
#
# Example tree output:
#   PID USER     CPU% MEM% Command
#     1 root      0.0  0.1 systemd
#   ├─ 512 root   0.0  0.1 ├─ sshd
#   │  └─ 891 john 0.1  0.2 │ └─ sshd: john@pts/0
#   │     └─ 892 john 0.0 0.1 │    └─ bash
#   ├─ 210 root   0.0  0.2 ├─ nginx: master process
#   │  ├─ 211 www-data 0.3 0.6 │ ├─ nginx: worker process
#   │  └─ 212 www-data 0.2 0.5 │ └─ nginx: worker process
```

> 💡 **Tip:** Killing a parent process in tree view will also terminate all its children.

---

### 4.9 Customizing `htop` (F2 Setup)

```bash
# 1. Inside htop, press F2 (or 'S')
# 2. Navigate with arrow keys through:
#
#    - Meters: Add/remove CPU, memory, disk, network meters
#    - Display options: Show threads, highlight changes, etc.
#    - Colors: Choose color schemes
#    - Columns: Add/remove/reorder process list columns
#
# 3. Press F10 to save settings
```

**Recommended Customizations:**
- Add `IO_READ_RATE` and `IO_WRITE_RATE` columns to spot disk-heavy processes
- Enable "Highlight program basename" for cleaner command display
- Add "Load average" and "Uptime" meters if not visible

---

## 5. Lab Part 3: `top` vs `htop` — Side by Side

| Feature | `top` | `htop` |
|---------|-------|--------|
| **Pre-installed** | ✅ Yes, on all Linux systems | ❌ Must install separately |
| **Color display** | ⚠️ Optional (`z` key) | ✅ Always color-coded |
| **Mouse support** | ❌ No | ✅ Yes |
| **Scroll process list** | ❌ Limited | ✅ Full scrolling |
| **Kill process** | ⚠️ Manual PID entry | ✅ Interactive selection + F9 |
| **Tree view** | ✅ Yes (`V` key) | ✅ Yes (`F5` key) |
| **Per-CPU bars** | ⚠️ Toggle with `1` | ✅ Visual bars by default |
| **Search by name** | ⚠️ Locate (`L`) | ✅ Live search (`F3`) |
| **Filter by user** | ✅ Yes (`u` key) | ✅ Yes (`u` key) |
| **Configuration** | Save with `W` | Save with `F10` in setup |
| **Batch mode** | ✅ Yes (`-b`) | ❌ Interactive only |
| **Best for** | Minimal systems, scripting | Daily interactive monitoring |

> 💡 **Rule of Thumb:** Use `top` when `htop` isn't available (e.g., rescue mode, minimal containers). Install `htop` everywhere else for a superior interactive experience.

---

## 6. Lab Part 4: Real-World Troubleshooting Scenarios

### Scenario 1: 🔥 System is Slow — Find the CPU Hog

```bash
# Method 1: Using top
 top
# Press 'P' to sort by CPU
# The top process is your culprit

# Method 2: Using htop
 htop
# Already sorted by CPU by default
# Select the process, press F9 to kill if needed

# Method 3: One-liner with ps
 ps aux --sort=-%cpu | head -10
```

### Scenario 2: 🧠 Running Out of Memory — Find the Memory Hog

```bash
# Method 1: Using top
 top
# Press 'M' to sort by memory

# Method 2: Using htop
 htop
# Press F6, select MEM%, press Enter

# Method 3: One-liner with ps
 ps aux --sort=-%mem | head -10
```

### Scenario 3: 🌳 Find Which Parent Spawned Too Many Children

```bash
# Using htop tree view
 htop
# Press F5 for tree view
# Look for parents with many branches

# Using ps
 ps auxf | head -30
```

### Scenario 4: 🔍 Investigate a Specific Process

```bash
# Find the PID first
 pgrep -a firefox

# Deep inspect with htop
 htop -p $(pgrep firefox)

# Check open files
 lsof -p $(pgrep firefox)

# Check what it's doing (inside htop)
# Select process → press 's' for strace
# Select process → press 'l' for lsof
```

### Scenario 5: 🛑 Kill a Runaway Process

```bash
# Step 1: Identify the process
 htop

# Step 2: Select it, press F9

# Step 3: Choose signal 15 (SIGTERM) — press Enter

# Step 4: Wait 5-10 seconds

# Step 5: If still running, repeat with signal 9 (SIGKILL)
```

### Scenario 6: ⚖️ Deprioritize a Background Task

```bash
# You have a backup script consuming too much CPU
# You want it to run, but not interfere with web server

 htop
# Select the backup process
# Press F8 repeatedly to raise nice value to +10 or +15
# The process will still run but yield CPU to higher-priority tasks
```

---

## 7. Quick Reference Cheat Sheet

### `top` Cheat Sheet

```
Launch:     top
Quit:       q
Help:       h or ?
Sort CPU:   P
Sort MEM:   M
Sort Time:  T
Toggle CPU: 1
Toggle Color: z
Full Path:  c
Tree View:  V
Kill:       k → enter PID → enter signal
Renice:     r → enter PID → enter nice value
Filter User: u → enter username
Batch Mode: top -bn1
```

### `htop` Cheat Sheet

```
Launch:     htop
Quit:       q or F10
Help:       F1 or h
Setup:      F2 or S
Search:     F3 or /
Filter:     F4 or Tree:       F5 or t
Sort:       F6 or < >
Nice -:     F7 or ]    (increase priority, root only)
Nice +:     F8 or [    (decrease priority)
Kill:       F9 or k
Tag:        Space
Untag All:  U
User Filter: u
Strace:     s
Lsof:       l
Follow:     F
Pause:      Z
```

### Common Signals Reference

| Signal | Number | Action | When to Use |
|--------|--------|--------|-------------|
| `SIGHUP` | 1 | Hang up / reload | Reload config without restart |
| `SIGINT` | 2 | Interrupt (Ctrl+C) | Graceful stop from terminal |
| `SIGQUIT` | 3 | Quit + dump core | Debug crash |
| `SIGKILL` | 9 | Force kill | Last resort, no cleanup |
| `SIGTERM` | 15 | Graceful termination | **Default, always try first** |
| `SIGCONT` | 18 | Continue | Resume stopped process |
| `SIGSTOP` | 19 | Stop (pause) | Pause without terminating |

---

## 8. Further Reading

- [Linux `top` man page](https://man7.org/linux/man-pages/man1/top.1.html)
- [Linux `htop` man page](https://man7.org/linux/man-pages/man1/htop.1.html)
- [Linux `ps` man page](https://man7.org/linux/man-pages/man1/ps.1.html)
- [Understanding Linux Load Average](https://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html)
- [Linux Process Management](https://www.linux.com/training-tutorials/linux-process-management/)

---

## ✅ Lab Completion Checklist

- [ ] Launched `top` and read all header fields
- [ ] Toggled CPU view (`1`), color (`z`), and tree view (`V`) in top
- [ ] Sorted by CPU (`P`) and Memory (`M`) in top
- [ ] Killed a test process using `top` (use a safe process like `sleep 300`)
- [ ] Installed `htop` if not already present
- [ ] Launched `htop` and identified all meter colors
- [ ] Used F3 to search for a process by name
- [ ] Used F4 to filter the process list
- [ ] Used F5 to view process tree
- [ ] Used F6 to sort by different columns
- [ ] Changed a process priority with F7/F8
- [ ] Killed a test process using `htop` F9
- [ ] Customized htop display via F2 Setup
- [ ] Completed all 6 real-world troubleshooting scenarios

---

> 🎓 **Congratulations!** You now have a solid understanding of Linux process monitoring using both `top` and `htop`. These skills are fundamental for system administration, DevOps, and debugging performance issues.

---

*Lab created for GitHub Learning. Based on Linux process monitoring fundamentals.*
