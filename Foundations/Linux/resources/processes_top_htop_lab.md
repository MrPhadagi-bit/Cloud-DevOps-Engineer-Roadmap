
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

