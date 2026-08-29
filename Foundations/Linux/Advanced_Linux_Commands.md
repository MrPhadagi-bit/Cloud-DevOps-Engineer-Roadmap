#  Advanced Linux Commands

> A comprehensive guide to advanced Linux commands for system administrators, DevOps engineers, developers, and power users.
> 
---

## 1. File & Directory Operations

### `find` — Search Files & Directories
Search for files and directories based on name, size, permissions, modification time, and more.

```bash
# Find files by name
find /home -name "*.txt"

# Find files larger than 100MB
find / -type f -size +100M

# Find files modified in the last 7 days
find . -mtime -7

# Find and delete empty directories
find . -type d -empty -delete

# Find files by permission
find /var/www -perm 644

# Execute command on found files
find . -name "*.log" -exec gzip {} \;
```

### `locate` — Fast File Search
Uses a prebuilt database for lightning-fast file searches (requires `updatedb`).

```bash
# Update the locate database
sudo updatedb

# Search for a file
locate nginx.conf

# Case-insensitive search
locate -i "*.pdf"

# Count matching entries
locate -c passwd
```

### `tree` — Visual Directory Structure
Display directory contents in a tree-like format.

```bash
# Show directory tree
tree

# Limit depth to 2 levels
tree -L 2

# Show only directories
tree -d
```

### `ln` — Create Links
Create hard links and symbolic (soft) links.

```bash
# Create a symbolic link
ln -s /usr/bin/python3 python

# Create a hard link
ln file.txt hardlink.txt

# Force overwrite existing link
ln -sf /new/path linkname
```

### `stat` — File Metadata
Display detailed file information including timestamps, size, and inode.

```bash
# Show file metadata
stat file.txt

# Custom format output
stat -c '%A %n' *

# Show filesystem info
stat -f /dev/sda1
```

### `shred` — Securely Delete Files
Overwrite files multiple times before deletion to prevent recovery.

```bash
# Securely delete a file
shred -u confidential.txt

# Overwrite 25 times (default is 3)
shred -n 25 -u secret.doc
```

---

## 2. File Permissions & Ownership

### `chmod` — Change File Permissions
Modify read, write, and execute permissions for files and directories.

```bash
# Add execute permission
chmod +x script.sh

# Set permissions numerically (rwxr-xr-x)
chmod 755 deploy.sh

# Recursive permission change
chmod -R 644 /var/www/html

# Set sticky bit on directory
chmod +t /shared_folder
```

**Permission Numbers:**
| Number | Permission | Symbol |
|--------|-----------|--------|
| 7 | Read, Write, Execute | `rwx` |
| 6 | Read, Write | `rw-` |
| 5 | Read, Execute | `r-x` |
| 4 | Read Only | `r--` |
| 0 | No Permission | `---` |

### `chown` — Change Ownership
Change the owner and/or group of a file or directory.

```bash
# Change owner
chown ubuntu file.txt

# Change owner and group
chown ubuntu:developers file.txt

# Recursive ownership change
chown -R www-data:www-data /var/www
```

### `chgrp` — Change Group
Change the group ownership of a file.

```bash
# Change group
chgrp developers script.sh
```

### `umask` — Default Permissions
Set default permissions for newly created files and directories.

```bash
# Show current umask
umask

# Set umask (files: 644, dirs: 755)
umask 022
```

---

## 3. Text Processing & Manipulation

### `grep` — Search Text Patterns
Search for patterns within files using regular expressions.

```bash
# Search for a string in a file
grep "ERROR" app.log

# Case-insensitive search
grep -i "failed" auth.log

# Search recursively in directory
grep -r "TODO" /home/user/projects

# Show line numbers
grep -n "pattern" file.txt

# Invert match (show lines that DON'T match)
grep -v "DEBUG" log.txt

# Count matches
grep -c "success" results.txt

# Search with regex
grep -E "^[0-9]{3}-[0-9]{2}-[0-9]{4}$" data.txt
```

### `sed` — Stream Editor
Perform text transformations on an input stream.

```bash
# Replace text in a file
sed 's/http/https/g' config.txt

# Replace only the 2nd occurrence per line
sed 's/unix/linux/2' file.txt

# Delete lines matching a pattern
sed '/^#/d' config.conf

# Delete empty lines
sed '/^$/d' file.txt

# Replace in-place (edit file directly)
sed -i 's/old/new/g' file.txt

# Print lines 5-10
sed -n '5,10p' file.txt
```

### `awk` — Pattern Scanning & Processing
Powerful text processing tool for data extraction and reporting.

```bash
# Print first column
awk '{print $1}' access.log

# Print multiple columns
awk '{print $1, $4, $7}' data.txt

# Filter lines where column 3 > 100
awk '$3 > 100 {print $0}' scores.txt

# Calculate sum of a column
awk '{sum += $2} END {print sum}' numbers.txt

# Use custom delimiter
awk -F: '{print $1}' /etc/passwd

# Format output
awk '{printf "%-10s %s\n", $1, $2}' users.txt
```

### `cut` — Extract Columns
Remove sections from each line of files.

```bash
# Extract first field (delimiter: colon)
cut -d: -f1 /etc/passwd

# Extract characters 1-10
cut -c1-10 file.txt

# Extract multiple fields
cut -d, -f1,3,5 data.csv
```

### `sort` — Sort Lines
Sort lines of text files.

```bash
# Sort alphabetically
sort names.txt

# Sort numerically
sort -n numbers.txt

# Sort in reverse order
sort -r file.txt

# Sort by column
sort -k3 -n data.txt

# Remove duplicates
sort -u file.txt
```

### `uniq` — Report/Filter Repeated Lines
Filter adjacent matching lines from input.

```bash
# Remove duplicate lines (requires sorted input)
sort file.txt | uniq

# Count occurrences of each line
sort file.txt | uniq -c

# Show only duplicate lines
sort file.txt | uniq -d

# Show only unique lines
sort file.txt | uniq -u
```

### `tr` — Translate/Delete Characters
Translate or delete characters from standard input.

```bash
# Convert lowercase to uppercase
cat file.txt | tr 'a-z' 'A-Z'

# Delete specific characters
cat file.txt | tr -d '\n'

# Squeeze repeated characters
echo "hello    world" | tr -s ' '
```

### `xargs` — Build & Execute Commands
Convert input from standard input into arguments for a command.

```bash
# Find and delete files
find . -name "*.tmp" | xargs rm

# Handle filenames with spaces
find . -name "*.txt" -print0 | xargs -0 cat

# Run multiple parallel processes
seq 1 10 | xargs -P 4 -I {} echo "Processing {}"

# Pass arguments to a command
cat urls.txt | xargs -I {} curl -s {}
```

### `paste` — Merge Lines
Merge lines of files side by side.

```bash
# Merge two files line by line
paste file1.txt file2.txt

# Merge with custom delimiter
paste -d',' names.txt ages.txt
```

### `diff` — Compare Files
Compare files line by line.

```bash
# Compare two files
diff old.conf new.conf

# Side-by-side comparison
diff -y file1.txt file2.txt

# Show differences in unified format
diff -u file1.txt file2.txt
```

### `wc` — Word/Line/Character Count
Count lines, words, and bytes in files.

```bash
# Count lines
wc -l file.txt

# Count words
wc -w file.txt

# Count characters
wc -c file.txt

# All counts
wc file.txt
```

---

## 4. Process Management

### `ps` — Process Status
Report a snapshot of current processes.

```bash
# Show all processes
ps aux

# Show processes for a specific user
ps -u ubuntu

# Show process tree
ps auxf

# Find process by name
ps aux | grep nginx

# Show specific columns
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head
```

### `top` / `htop` — Process Viewer
Display real-time system processes and resource usage.

```bash
# Launch top
top

# Sort by memory usage (press Shift+M in top)
# Sort by CPU usage (press Shift+P in top)

# Launch htop (more user-friendly)
htop
```

**Top Interactive Commands:**
| Key | Action |
|-----|--------|
| `k` | Kill a process |
| `r` | Renice a process |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `q` | Quit |

### `kill` / `killall` — Terminate Processes
Send signals to processes.

```bash
# Kill process by PID
kill 1234

# Force kill (SIGKILL)
kill -9 1234

# Kill process by name
killall firefox

# Send specific signal
kill -HUP 1234
```

**Common Signals:**
| Signal | Number | Description |
|--------|--------|-------------|
| SIGHUP | 1 | Hang up / Reload config |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGKILL | 9 | Force kill |
| SIGTERM | 15 | Graceful termination |

### `nice` / `renice` — Process Priority
Set or change process priority.

```bash
# Start process with low priority
nice -n 10 python app.py

# Change priority of running process
renice 10 -p 1234

# Give highest priority (requires root)
sudo nice -n -5 ./important_task
```

### `bg` / `fg` / `jobs` — Job Control
Manage background and foreground jobs.

```bash
# List background jobs
jobs

# Suspend current process (Ctrl+Z)
# Then send to background
bg

# Bring background job to foreground
fg %1

# Start process in background
python script.py &
```

### `lsof` — List Open Files
List information about files opened by processes.

```bash
# List all open files
lsof

# List files opened by specific process
lsof -p 1234

# List processes using a specific file
lsof /var/log/syslog

# List open network connections
lsof -i

# List processes using a specific port
lsof -i :8080
```

### `strace` — Trace System Calls
Trace system calls and signals of a process.

```bash
# Trace a command
strace ls

# Trace a running process
strace -p 1234

# Trace file operations only
strace -e trace=file cat /etc/passwd

# Save output to file
strace -o output.txt ./program
```

---

## 5. System Monitoring & Information

### `uptime` — System Uptime
Show how long the system has been running.

```bash
# Show uptime and load average
uptime

# Pretty output
uptime -p
```

**Load Average Interpretation:**
- **1.0** on single-core = 100% CPU utilization
- Values above number of cores = system is overloaded
- Ideal: below 0.7 × number of CPU cores

### `free` — Memory Usage
Display amount of free and used memory.

```bash
# Show memory in human-readable format
free -h

# Show total, used, and free memory
free -m

# Continuous monitoring (every 2 seconds)
free -s 2
```

### `df` — Disk Space Usage
Report file system disk space usage.

```bash
# Show disk usage in human-readable format
df -h

# Show specific filesystem
df -h /

# Show inode usage
df -i

# Show all filesystems including pseudo
df -a
```

### `du` — Directory Disk Usage
Estimate file space usage.

```bash
# Show size of current directory
du -sh .

# Show sizes of all subdirectories (1 level deep)
du -h --max-depth=1

# Sort by size
du -ah | sort -h

# Exclude certain directories
du -sh --exclude='*.log' /var
```

### `vmstat` — Virtual Memory Statistics
Report virtual memory, processes, CPU activity.

```bash
# Show statistics
vmstat

# Show every 2 seconds, 5 times
vmstat 2 5

# Show disk statistics
vmstat -d
```

### `iostat` — I/O Statistics
Report CPU and I/O statistics.

```bash
# Show CPU and disk I/O
iostat

# Extended statistics every 2 seconds
iostat -x 2

# Show only disk statistics
iostat -d
```

### `dmesg` — Kernel Ring Buffer
Print kernel message buffer.

```bash
# Show all kernel messages
dmesg

# Show human-readable timestamps
dmesg -T

# Follow new messages
dmesg -w

# Filter for errors
dmesg | grep -i error
```

### `journalctl` — Systemd Logs
Query the systemd journal.

```bash
# Show all logs
journalctl

# Show logs since last boot
journalctl -b

# Follow logs in real-time
journalctl -f

# Show logs for a specific service
journalctl -u nginx

# Show logs from last hour
journalctl --since "1 hour ago"

# Show kernel logs only
journalctl -k
```

### `uname` — System Information
Print system information.

```bash
# Show all information
uname -a

# Show kernel name
uname -s

# Show kernel release
uname -r

# Show machine hardware name
uname -m
```

### `lscpu` — CPU Information
Display CPU architecture information.

```bash
# Show CPU info
lscpu

# Show in parseable format
lscpu -p
```

### `lsblk` — Block Devices
List information about block devices.

```bash
# Show block devices
lsblk

# Show filesystem info
lsblk -f

# Show size in bytes
lsblk -b
```

### `dmidecode` — Hardware Information
Dump DMI (SMBIOS) table contents.

```bash
# Show all hardware info
sudo dmidecode

# Show memory info
sudo dmidecode -t memory

# Show processor info
sudo dmidecode -t processor

# Show system info
sudo dmidecode -t system
```

---

## 6. Networking Commands

### `ip` — Modern Networking Tool
Show/manipulate routing, network devices, interfaces, and tunnels.

```bash
# Show IP addresses
ip addr

# Show specific interface
ip addr show eth0

# Show routing table
ip route

# Show network links
ip link

# Add IP address to interface
sudo ip addr add 192.168.1.100/24 dev eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

### `ss` — Socket Statistics
Investigate sockets (replacement for netstat).

```bash
# Show all sockets
ss

# Show listening TCP sockets
ss -ltn

# Show all TCP/UDP sockets with process info
ss -tulpn

# Show connected sockets
ss -t state established

# Show memory usage of sockets
ss -m
```

### `netstat` — Network Statistics
Display network connections, routing tables, interface statistics.

```bash
# Show all listening ports
netstat -tulpn

# Show routing table
netstat -r

# Show network interface statistics
netstat -i

# Show all connections
netstat -an
```

### `ifconfig` — Interface Configuration
Configure network interfaces (legacy, use `ip` instead).

```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Enable/disable interface
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

### `ping` — Test Connectivity
Send ICMP echo requests to test network reachability.

```bash
# Ping a host
ping google.com

# Ping with count limit
ping -c 4 google.com

# Ping with interval
ping -i 2 google.com
```

### `traceroute` / `tracepath` — Trace Route
Print the route packets take to a network host.

```bash
# Trace route to host
traceroute google.com

# Trace without resolving hostnames
traceroute -n google.com

# Use ICMP instead of UDP
traceroute -I google.com
```

### `dig` — DNS Lookup
DNS lookup utility.

```bash
# Query DNS for A record
dig google.com

# Query specific record type
dig google.com MX

# Short output
dig +short google.com

# Trace DNS resolution
dig +trace google.com

# Reverse DNS lookup
dig -x 8.8.8.8
```

### `nslookup` — Query DNS
Query Internet name servers interactively.

```bash
# Lookup A record
nslookup google.com

# Query specific DNS server
nslookup google.com 8.8.8.8

# Query MX records
nslookup -query=MX google.com
```

### `host` — DNS Lookup
Simple DNS lookup utility.

```bash
# Lookup host
host google.com

# Reverse lookup
host 8.8.8.8

# Query all records
host -a google.com
```

### `curl` — Transfer Data
Transfer data from or to a server.

```bash
# Download a file
curl -O https://example.com/file.zip

# Follow redirects
curl -L https://example.com

# Show headers only
curl -I https://example.com

# POST data
curl -X POST -d "name=value" https://api.example.com

# Save output to file
curl -o output.html https://example.com

# Send custom headers
curl -H "Authorization: Bearer token" https://api.example.com
```

### `wget` — Download Files
Non-interactive network downloader.

```bash
# Download a file
wget https://example.com/file.zip

# Download in background
wget -b https://example.com/large-file.zip

# Resume interrupted download
wget -c https://example.com/file.zip

# Download recursively
wget -r -l 5 https://example.com

# Mirror a website
wget --mirror --convert-links --adjust-extension --page-requisites --no-parent https://example.com
```

### `scp` — Secure Copy
Copy files between hosts over SSH.

```bash
# Copy local file to remote server
scp file.txt user@server:/home/user/

# Copy remote file to local
scp user@server:/home/user/file.txt ./

# Copy directory recursively
scp -r folder/ user@server:/home/user/

# Copy with specific port
scp -P 2222 file.txt user@server:/home/user/
```

### `rsync` — Remote Sync
Fast, versatile file copying tool.

```bash
# Sync local directories
rsync -av /source/ /destination/

# Sync to remote server
rsync -avz /local/path/ user@server:/remote/path/

# Show progress
rsync -av --progress /source/ /destination/

# Delete files not in source
rsync -av --delete /source/ /destination/

# Exclude files
rsync -av --exclude='*.tmp' /source/ /destination/
```

**Rsync Options:**
| Option | Description |
|--------|-------------|
| `-a` | Archive mode (preserves permissions, timestamps, etc.) |
| `-v` | Verbose |
| `-z` | Compress during transfer |
| `--delete` | Delete files in destination not in source |
| `--progress` | Show progress during transfer |

### `nc` / `netcat` — Network Swiss Army Knife
Read/write network connections using TCP or UDP.

```bash
# Test port connectivity
nc -zv google.com 80

# Listen on a port
nc -l 8080

# Create a simple chat server
nc -l 1234

# Send a file
nc -w 3 destination_host 1234 < file.txt

# Port scan
nc -zv localhost 1-1000
```

### `nmap` — Network Scanner
Network discovery and security auditing.

```bash
# Scan a host
nmap 192.168.1.1

# Scan a network
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 192.168.1.1

# Aggressive scan
nmap -A 192.168.1.1

# Stealth scan
nmap -sS 192.168.1.1
```

### `tcpdump` — Packet Analyzer
Capture and analyze network packets.

```bash
# Capture packets on interface
sudo tcpdump -i eth0

# Capture packets to file
sudo tcpdump -w capture.pcap -i eth0

# Read from capture file
tcpdump -r capture.pcap

# Filter by host
sudo tcpdump host 192.168.1.1

# Filter by port
sudo tcpdump port 80

# Show packet contents
sudo tcpdump -A -i eth0
```

### `iptables` — Firewall Rules
Administration tool for IPv4 packet filtering and NAT.

```bash
# List all rules
sudo iptables -L

# List with line numbers
sudo iptables -L --line-numbers

# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Block an IP
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Save rules
sudo iptables-save > rules.txt

# Restore rules
sudo iptables-restore < rules.txt
```

---

## 7. Disk & Filesystem Management

### `fdisk` / `cfdisk` — Partition Tables
Manipulate disk partition tables.

```bash
# List partitions
sudo fdisk -l

# Edit partition table
sudo fdisk /dev/sda

# Interactive partition editor
sudo cfdisk /dev/sda
```

### `mkfs` — Create Filesystem
Build a Linux filesystem on a device.

```bash
# Create ext4 filesystem
sudo mkfs.ext4 /dev/sda1

# Create xfs filesystem
sudo mkfs.xfs /dev/sda1

# Create NTFS filesystem
sudo mkfs.ntfs /dev/sda1
```

### `fsck` — Filesystem Check
Check and repair a Linux filesystem.

```bash
# Check filesystem
sudo fsck /dev/sda1

# Force check even if clean
sudo fsck -f /dev/sda1

# Check all filesystems
sudo fsck -A
```

### `mount` / `umount` — Mount/Unmount
Mount and unmount filesystems.

```bash
# Mount a device
sudo mount /dev/sda1 /mnt

# Mount with specific filesystem
sudo mount -t ext4 /dev/sda1 /mnt

# Mount ISO file
sudo mount -o loop file.iso /mnt/iso

# Unmount
sudo umount /mnt

# Show mounted filesystems
mount | grep ext4
```

### `dd` — Convert & Copy Files
Copy and convert files at the block level.

```bash
# Create bootable USB
sudo dd if=image.iso of=/dev/sdb bs=4M status=progress

# Backup MBR
sudo dd if=/dev/sda of=mbr_backup.bin bs=512 count=1

# Create empty file of specific size
dd if=/dev/zero of=file.img bs=1M count=1024

# Clone disk
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress
```

 **Warning:** `dd` is extremely powerful. Double-check `if=` (input file) and `of=` (output file) before executing!

### `parted` — Partition Editor
Partition manipulation program.

```bash
# Start parted
sudo parted /dev/sda

# Print partition table
sudo parted /dev/sda print

# Resize partition
sudo parted /dev/sda resizepart 1 100%
```

---

## 8. User & Group Management

### `useradd` / `usermod` / `userdel` — User Management
Create, modify, and delete user accounts.

```bash
# Create a new user
sudo useradd -m -s /bin/bash username

# Create user with home directory and password
sudo useradd -m username
sudo passwd username

# Modify user (add to sudo group)
sudo usermod -aG sudo username

# Delete user
sudo userdel username

# Delete user and home directory
sudo userdel -r username
```

### `groupadd` / `groupmod` / `groupdel` — Group Management
Create, modify, and delete groups.

```bash
# Create a group
sudo groupadd developers

# Delete a group
sudo groupdel developers

# Modify group
sudo groupmod -n newname oldname
```

### `passwd` — Password Management
Change user passwords.

```bash
# Change your password
passwd

# Change another user's password
sudo passwd username

# Lock user account
sudo passwd -l username

# Unlock user account
sudo passwd -u username

# Force password change on next login
sudo passwd -e username
```

### `chage` — Password Aging
View and change user password expiry information.

```bash
# Show password aging info
sudo chage -l username

# Set password to expire in 30 days
sudo chage -M 30 username

# Set account expiration date
sudo chage -E 2026-12-31 username
```

### `su` — Switch User
Change to another user account.

```bash
# Switch to root
su -

# Switch to specific user
su - username

# Execute command as another user
su - username -c "whoami"
```

### `sudo` — Execute as Superuser
Execute commands with superuser privileges.

```bash
# Run command as root
sudo apt update

# Switch to root shell
sudo -i

# Run command as specific user
sudo -u www-data whoami

# Edit file with root privileges
sudo nano /etc/hosts
```

### `who` / `w` / `whoami` — User Information
Show who is logged in and what they are doing.

```bash
# Show logged-in users
who

# Show detailed user activity
w

# Show current user
whoami

# Show user ID and groups
id
```

### `last` — Login History
Show listing of last logged in users.

```bash
# Show last logins
last

# Show last logins for specific user
last username

# Show failed login attempts
lastb
```

### `wall` — Broadcast Message
Send a message to all logged-in users.

```bash
# Send message to all users
wall "System will reboot in 5 minutes!"

# Send message from file
wall < message.txt
```

### `write` — Send Message to User
Send a message to another user.

```bash
# Send message to user
write username

# Send message to specific terminal
write username tty1
```

### `mesg` — Control Write Access
Control write access to your terminal.

```bash
# Allow messages
mesg y

# Disallow messages
mesg n

# Check current status
mesg
```

---

## 9. Compression & Archiving

### `tar` — Tape Archive
Create and extract archive files.

```bash
# Create tar archive
tar -cvf archive.tar /path/to/files

# Create compressed tar.gz archive
tar -czvf archive.tar.gz /path/to/files

# Create tar.bz2 archive
tar -cjvf archive.tar.bz2 /path/to/files

# Extract tar archive
tar -xvf archive.tar

# Extract tar.gz archive
tar -xzvf archive.tar.gz

# List contents of archive
tar -tvf archive.tar

# Extract specific file from archive
tar -xvf archive.tar file.txt
```

**Tar Options:**
| Option | Description |
|--------|-------------|
| `-c` | Create archive |
| `-x` | Extract archive |
| `-v` | Verbose |
| `-f` | Specify filename |
| `-z` | Compress with gzip |
| `-j` | Compress with bzip2 |
| `-t` | List contents |

### `gzip` / `gunzip` — Gzip Compression
Compress and decompress files using Lempel-Ziv coding.

```bash
# Compress a file
gzip file.txt

# Compress with best compression
gzip -9 file.txt

# Decompress a file
gunzip file.txt.gz

# Keep original file
gzip -k file.txt
```

### `zip` / `unzip` — ZIP Archives
Package and compress files.

```bash
# Create zip archive
zip archive.zip file1.txt file2.txt

# Zip a directory recursively
zip -r archive.zip folder/

# Extract zip archive
unzip archive.zip

# Extract to specific directory
unzip archive.zip -d /destination/

# List contents without extracting
unzip -l archive.zip
```

### `7z` — 7-Zip Archives
High compression ratio file archiver.

```bash
# Create 7z archive
7z a archive.7z files/

# Extract 7z archive
7z x archive.7z

# List contents
7z l archive.7z
```

---

## 10. Package Management

### `apt` / `apt-get` — Debian/Ubuntu
Package management for Debian-based distributions.

```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade

# Full upgrade (removes obsolete packages)
sudo apt full-upgrade

# Install a package
sudo apt install nginx

# Remove a package
sudo apt remove nginx

# Remove package and configuration
sudo apt purge nginx

# Clean up
sudo apt autoremove
sudo apt clean

# Search for packages
apt search nginx

# Show package info
apt show nginx
```

### `dpkg` — Debian Package
Low-level package manager for Debian.

```bash
# Install .deb package
sudo dpkg -i package.deb

# Remove package
sudo dpkg -r package

# List installed packages
dpkg -l

# Show package details
dpkg -s nginx
```

### `yum` / `dnf` — RHEL/CentOS/Fedora
Package management for Red Hat-based distributions.

```bash
# Install package (yum)
sudo yum install nginx

# Install package (dnf)
sudo dnf install nginx

# Update packages
sudo yum update
sudo dnf update

# Remove package
sudo yum remove nginx
sudo dnf remove nginx

# Search packages
yum search nginx
dnf search nginx

# Show package info
yum info nginx
dnf info nginx
```

### `rpm` — RPM Package Manager
Low-level package manager for Red Hat-based systems.

```bash
# Install RPM package
sudo rpm -ivh package.rpm

# Remove package
sudo rpm -e package

# Query installed packages
rpm -qa

# Verify package
rpm -V package
```

### `snap` — Snap Packages
Universal Linux package system.

```bash
# Install snap package
sudo snap install code

# Remove snap package
sudo snap remove code

# List installed snaps
snap list

# Refresh all snaps
sudo snap refresh
```

---

## 11. Job Scheduling

### `cron` / `crontab` — Scheduled Tasks
Schedule commands to run periodically.

```bash
# Edit crontab
crontab -e

# List crontab entries
crontab -l

# Remove crontab
crontab -r

# Edit root crontab
sudo crontab -e
```

**Crontab Format:**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Common Examples:**
```bash
# Run every minute
* * * * * /path/to/script.sh

# Run every hour at minute 0
0 * * * * /path/to/script.sh

# Run daily at 2:30 AM
30 2 * * * /path/to/script.sh

# Run weekly on Sunday at 3:00 AM
0 3 * * 0 /path/to/script.sh

# Run monthly on the 1st at midnight
0 0 1 * * /path/to/script.sh

# Run every 5 minutes
*/5 * * * * /path/to/script.sh

# Run Monday-Friday at 9 AM
0 9 * * 1-5 /path/to/script.sh
```

### `at` — One-Time Scheduled Tasks
Schedule commands to run once at a specific time.

```bash
# Schedule a job for later
echo "/path/to/script.sh" | at 14:00

# Schedule with date
echo "backup.sh" | at 2:00 AM tomorrow

# List pending jobs
atq

# Remove a job
atrm 1

# Show job details
at -c 1
```

---

## 12. Bash Shortcuts & Tips

### Navigation Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + A` | Move cursor to beginning of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + B` | Move back one character |
| `Ctrl + F` | Move forward one character |
| `Alt + B` | Move back one word |
| `Alt + F` | Move forward one word |

### Editing Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + U` | Delete from cursor to beginning of line |
| `Ctrl + K` | Delete from cursor to end of line |
| `Ctrl + W` | Delete word before cursor |
| `Ctrl + Y` | Paste last deleted text |
| `Ctrl + L` | Clear screen |
| `Ctrl + C` | Cancel current command |
| `Ctrl + Z` | Suspend current process |

### History Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + R` | Search command history |
| `↑ / ↓` | Browse command history |
| `!!` | Repeat last command |
| `!n` | Execute command number n from history |
| `!string` | Execute last command starting with 'string' |
| `!$` | Last argument of previous command |
| `!*` | All arguments of previous command |

### Useful Bash Tips

```bash
# Run previous command with sudo
sudo !!

# Create multiple directories at once
mkdir -p project/{src,tests,docs}/{2025,2026}

# Brace expansion
echo file{1..5}.txt
# Output: file1.txt file2.txt file3.txt file4.txt file5.txt

# Command substitution
files=$(ls /var/log)
echo "Files: $files"

# Process substitution
diff <(sort file1.txt) <(sort file2.txt)

# Background process
long_running_command &

# Redirect stdout and stderr
command > output.txt 2>&1

# Discard output
command > /dev/null 2>&1

# Here document
cat << EOF > config.txt
key1=value1
key2=value2
EOF
```

---

## 13. Dangerous Commands to Avoid

>  **WARNING:** These commands can cause irreversible damage. Use with extreme caution!

| Command | Risk | Safer Alternative |
|---------|------|-------------------|
| `rm -rf /` | Deletes entire filesystem | Always verify path: `rm -rf ./path` |
| `rm -rf /*` | Deletes all files from root | Use `rm -i` for confirmation |
| `:(){ :|:& };:` | Fork bomb — crashes system | Never run unknown shell functions |
| `mkfs.ext4 /dev/sda` | Formats entire disk | Double-check device with `lsblk` |
| `dd if=/dev/zero of=/dev/sda` | Overwrites entire disk | Verify `of=` target carefully |
| `chmod -R 777 /` | Breaks system permissions | Use `chmod 755` or `chmod 644` |
| `mv / /dev/null` | Moves root to null | Never move system directories |
| `> /dev/sda` | Overwrites disk with null | Be careful with redirection operators |

### Safety Tips

1. **Always use absolute paths** when running destructive commands
2. **Use `rm -i`** for interactive deletion confirmation
3. **Test commands with `echo`** before executing: `echo rm *.txt`
4. **Use `trash-cli`** instead of `rm` for recoverable deletion
5. **Make backups** before running system-level commands
6. **Use `set -u`** in scripts to catch undefined variables

---

##  Additional Resources

- [Linux Commands Cheat Sheet - GeeksforGeeks](https://www.geeksforgeeks.org/linux-unix/linux-commands-cheat-sheet/)
- [Advanced Linux Commands Cheat Sheet - Red Hat](https://developers.redhat.com/cheat-sheets/advanced-linux-commands)
- [Linux Man Pages](https://man7.org/linux/man-pages/)
- [Explain Shell](https://explainshell.com/) — Visualize command explanations

---

##  Contributing

Feel free to contribute by submitting a pull request. Suggestions for new commands, corrections, or improvements are always welcome!

---

##  License

This guide is provided as-is for educational purposes. Use the commands at your own risk.

---

>  **Pro Tip:** Bookmark this page and refer back to it as you level up your Linux skills. Practice these commands in a safe environment like a VM or Docker container before using them on production systems!
