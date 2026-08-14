# Linux Users & Groups — Complete Guide

> A comprehensive guide to understanding and managing Linux users and groups, permissions, and access control. Based on the [Linux for Programmers #3](https://youtu.be/b-9j2jiCOEA) series by Tech With Tim & Linode.

---

## Table of Contents

- [Introduction](#introduction)
- [Understanding Linux Users](#understanding-linux-users)
  - [Types of Users](#types-of-users)
  - [User IDs (UID)](#user-ids-uid)
  - [The `/etc/passwd` File](#the-etcpasswd-file)
  - [The `/etc/shadow` File](#the-etcshadow-file)
- [Managing Users](#managing-users)
  - [Creating Users](#creating-users)
  - [Setting/Changing Passwords](#settingchanging-passwords)
  - [Switching Between Users](#switching-between-users)
  - [Modifying Users](#modifying-users)
  - [Deleting Users](#deleting-users)
- [Understanding Linux Groups](#understanding-linux-groups)
  - [Primary vs. Secondary Groups](#primary-vs-secondary-groups)
  - [Group IDs (GID)](#group-ids-gid)
  - [The `/etc/group` File](#the-etcgroup-file)
- [Managing Groups](#managing-groups)
  - [Creating Groups](#creating-groups)
  - [Adding Users to Groups](#adding-users-to-groups)
  - [Removing Users from Groups](#removing-users-from-groups)
  - [Deleting Groups](#deleting-groups)
- [File Permissions & Ownership](#file-permissions--ownership)
  - [Understanding Permission Bits](#understanding-permission-bits)
  - [Changing Permissions with `chmod`](#changing-permissions-with-chmod)
  - [Changing Ownership with `chown`](#changing-ownership-with-chown)
  - [Changing Group with `chgrp`](#changing-group-with-chgrp)
  - [Special Permissions (SUID, SGID, Sticky Bit)](#special-permissions-suid-sgid-sticky-bit)
- [Sudo & Privilege Escalation](#sudo--privilege-escalation)
  - [The `sudoers` File](#the-sudoers-file)
  - [Configuring Sudo Access](#configuring-sudo-access)
- [Useful Commands Reference](#useful-commands-reference)
- [Best Practices](#best-practices)
- [Further Reading](#further-reading)

---

## Introduction

Linux is a **multi-user operating system**, meaning multiple users can access the system simultaneously, each with their own files, processes, and environment. The user and group system is the foundation of Linux security — it controls who can access what resources on the system.

> **Key Concept:** Everything in Linux is owned by a **user** and belongs to a **group**. Permissions are assigned based on these associations.

---

## Understanding Linux Users

### Types of Users

| User Type | UID Range | Description |
|-----------|-----------|-------------|
| **Root (Superuser)** | `0` | Has unlimited privileges. Can do anything on the system. |
| **System Users** | `1` – `999` | Created by the system for running services (e.g., `www-data`, `mysql`, `sshd`). |
| **Regular Users** | `1000+` | Human users created by the administrator. |

### User IDs (UID)

Every user on a Linux system has a unique **User ID (UID)**. The system uses UIDs internally to identify users, while usernames are for human convenience.

```bash
# View your current UID
echo $UID

# View all users with their UIDs
cat /etc/passwd | cut -d: -f1,3
```

### The `/etc/passwd` File

This file stores essential information about all user accounts on the system.

```bash
cat /etc/passwd
```

**Format (colon-separated fields):**

```
username:password:UID:GID:GECOS:home_directory:shell
```

| Field | Example | Description |
|-------|---------|-------------|
| `username` | `john` | Login name |
| `password` | `x` | Placeholder (`x` means password is in `/etc/shadow`) |
| `UID` | `1000` | User ID |
| `GID` | `1000` | Primary Group ID |
| `GECOS` | `John Doe` | Full name / comment field |
| `home_directory` | `/home/john` | User's home directory |
| `shell` | `/bin/bash` | Default login shell |

**Example entry:**
```
john:x:1000:1000:John Doe:/home/john:/bin/bash
```

### The `/etc/shadow` File

Stores encrypted user passwords and password aging information. **Only readable by root.**

```bash
sudo cat /etc/shadow
```

**Format:**
```
username:encrypted_password:last_change:min_age:max_age:warn:inactive:expire:reserved
```

---

## Managing Users

### Creating Users

#### `useradd` — Create a new user

```bash
# Basic user creation (creates user with defaults)
sudo useradd username

# Create user with home directory and default shell
sudo useradd -m -s /bin/bash username

# Create user with a specific home directory
sudo useradd -m -d /custom/home/dir username

# Create user with a comment (full name)
sudo useradd -m -c "John Smith" jsmith

# Create user with a specific UID
sudo useradd -m -u 1500 username

# Create user and add to specific groups
sudo useradd -m -G sudo,docker username

# Create a system user (no home, no login)
sudo useradd -r -s /usr/sbin/nologin serviceaccount
```

| Option | Description |
|--------|-------------|
| `-m` | Create home directory (`/home/username`) |
| `-s` | Specify login shell |
| `-d` | Specify custom home directory |
| `-c` | Add a comment (GECOS field) |
| `-u` | Specify custom UID |
| `-g` | Specify primary group |
| `-G` | Specify supplementary groups (comma-separated) |
| `-r` | Create a system account |
| `-e` | Set account expiration date (YYYY-MM-DD) |

#### `adduser` — Interactive alternative (Debian/Ubuntu)

```bash
# More user-friendly, interactive prompts
sudo adduser username
```

This command interactively asks for password, full name, and other details.

### Setting/Changing Passwords

```bash
# Set password for a user (as root)
sudo passwd username

# Change your own password
passwd

# Lock a user account (prevent login)
sudo passwd -l username

# Unlock a user account
sudo passwd -u username

# Force password change on next login
sudo passwd -e username

# Display password status
sudo passwd -S username
```

### Switching Between Users

#### `su` — Switch User

```bash
# Switch to root (requires root password)
su -

# Switch to another user (loads their environment)
su - username

# Switch to another user (keeps current environment)
su username

# Execute a single command as another user
su - username -c "whoami"
```

> **Note:** `su -` (with hyphen) simulates a full login, loading the target user's environment variables and starting in their home directory.

#### `sudo` — Execute as Superuser

```bash
# Run command as root
sudo command

# Switch to root shell
sudo -i

# Run command as a specific user
sudo -u username command

# Edit a file as root
sudo nano /etc/hosts
```

### Modifying Users

#### `usermod` — Modify a user account

```bash
# Change user's home directory and move contents
sudo usermod -m -d /new/home/dir username

# Change user's login shell
sudo usermod -s /bin/zsh username

# Change user's comment (full name)
sudo usermod -c "New Name" username

# Lock a user account
sudo usermod -L username

# Unlock a user account
sudo usermod -U username

# Change user's primary group
sudo usermod -g newprimarygroup username

# Add user to supplementary groups (appends)
sudo usermod -aG group1,group2 username

# ⚠️ Remove user from all supplementary groups and set new ones
sudo usermod -G group1,group2 username

# Change user's UID
sudo usermod -u 2000 username
```

> **⚠️ Important:** Always use `-a` (append) with `-G` to avoid removing existing group memberships!

### Deleting Users

#### `userdel` — Delete a user

```bash
# Delete user (keeps home directory and mail spool)
sudo userdel username

# Delete user and their home directory
sudo userdel -r username

# Force deletion even if user is logged in
sudo userdel -f username
```

| Option | Description |
|--------|-------------|
| `-r` | Remove home directory and mail spool |
| `-f` | Force removal (even if logged in) |

---

## Understanding Linux Groups

Groups are collections of users. They simplify permission management by allowing you to assign permissions to a group rather than individual users.

### Primary vs. Secondary Groups

| Type | Description | Example |
|------|-------------|---------|
| **Primary Group** | The default group assigned when a user creates files. Specified in `/etc/passwd`. | `john` creates a file → owned by group `john` |
| **Secondary (Supplementary) Groups** | Additional groups a user belongs to. Grants extra permissions. | `john` is in `sudo`, `docker`, `developers` |

Every user must have exactly **one primary group** and can belong to **zero or more secondary groups**.

### Group IDs (GID)

Every group has a unique **Group ID (GID)**.

```bash
# View your current GID
echo $GID

# View all groups with their GIDs
cat /etc/group | cut -d: -f1,3
```

### The `/etc/group` File

Stores information about all groups on the system.

```bash
cat /etc/group
```

**Format:**
```
groupname:password:GID:member1,member2,member3
```

**Example entry:**
```
developers:x:1001:john,jane,bob
```

| Field | Description |
|-------|-------------|
| `groupname` | Name of the group |
| `password` | Group password (rarely used, usually `x`) |
| `GID` | Group ID |
| `members` | Comma-separated list of usernames |

---

## Managing Groups

### Creating Groups

#### `groupadd` — Create a new group

```bash
# Create a basic group
sudo groupadd developers

# Create group with specific GID
sudo groupadd -g 1500 developers

# Create a system group
sudo groupadd -r sysgroup
```

| Option | Description |
|--------|-------------|
| `-g` | Specify custom GID |
| `-r` | Create a system group |

#### `addgroup` — Interactive alternative (Debian/Ubuntu)

```bash
sudo addgroup developers
```

### Adding Users to Groups

#### `usermod -aG` — Add user to supplementary groups

```bash
# Add user to a group (append mode - recommended)
sudo usermod -aG developers john

# Add user to multiple groups at once
sudo usermod -aG sudo,docker,developers john

# Change user's primary group
sudo usermod -g developers john
```

> **⚠️ Remember:** Without `-a`, `-G` will **replace** all existing supplementary groups!

#### `gpasswd` — Administer groups

```bash
# Add user to group
sudo gpasswd -a john developers

# Add multiple users to group
sudo gpasswd -M john,jane,bob developers

# Set group administrators
sudo gpasswd -A john developers
```

### Removing Users from Groups

```bash
# Remove user from a group using gpasswd
sudo gpasswd -d john developers

# Remove user from a group using usermod
# (You must specify ALL groups you want to keep)
sudo usermod -G remaininggroup1,remaininggroup2 john
```

### Deleting Groups

#### `groupdel` — Delete a group

```bash
# Delete a group
sudo groupdel developers

# Check if group exists
grep developers /etc/group
```

> **Note:** You cannot delete a group that is a user's primary group. Change the user's primary group first.

---

## File Permissions & Ownership

### Understanding Permission Bits

Every file and directory has permissions for three categories:

| Category | Symbol | Description |
|----------|--------|-------------|
| **Owner** | `u` | The user who owns the file |
| **Group** | `g` | The group that owns the file |
| **Others** | `o` | Everyone else |

Each category has three permission types:

| Permission | Symbol | File | Directory |
|------------|--------|------|-----------|
| **Read** | `r` (4) | View contents | List files |
| **Write** | `w` (2) | Modify contents | Create/delete files |
| **Execute** | `x` (1) | Run as program | Enter/access |

**View permissions with `ls -l`:**

```bash
ls -l file.txt
# -rw-r--r-- 1 john developers 1234 Aug 14 10:00 file.txt
#  |  |   |
#  |  |   └── Others permissions (r--)
#  |  └────── Group permissions (r--)
#  └───────── Owner permissions (rw-)
```

**Permission notation:**

| Notation | Binary | Decimal | Meaning |
|----------|--------|---------|---------|
| `---` | `000` | `0` | No permissions |
| `--x` | `001` | `1` | Execute only |
| `-w-` | `010` | `2` | Write only |
| `-wx` | `011` | `3` | Write + Execute |
| `r--` | `100` | `4` | Read only |
| `r-x` | `101` | `5` | Read + Execute |
| `rw-` | `110` | `6` | Read + Write |
| `rwx` | `111` | `7` | Read + Write + Execute |

### Changing Permissions with `chmod`

#### Numeric (Octal) Mode

```bash
# chmod [owner][group][others] filename

chmod 755 script.sh    # rwxr-xr-x
chmod 644 file.txt     # rw-r--r--
chmod 700 private.key  # rwx------
chmod 777 shared.txt   # rwxrwxrwx (⚠️ avoid!)
chmod 750 directory    # rwxr-x---
```

#### Symbolic Mode

```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for group
chmod g-w file.txt

# Add read permission for others
chmod o+r file.txt

# Set exact permissions
chmod u=rwx,g=rx,o=r file.txt

# Add read for all (user, group, others)
chmod a+r file.txt

# Recursively change permissions
chmod -R 755 /path/to/directory
```

| Symbol | Meaning |
|--------|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u + g + o) |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

### Changing Ownership with `chown`

```bash
# Change owner
sudo chown john file.txt

# Change owner and group
sudo chown john:developers file.txt

# Change only group (same as chgrp)
sudo chown :developers file.txt

# Recursively change ownership
sudo chown -R john:developers /path/to/directory

# Reference another file's ownership
sudo chown --reference=otherfile.txt targetfile.txt
```

### Changing Group with `chgrp`

```bash
# Change group ownership
sudo chgrp developers file.txt

# Recursively change group
sudo chgrp -R developers /path/to/directory
```

### Special Permissions (SUID, SGID, Sticky Bit)

| Special Bit | Numeric | Symbol | Description |
|-------------|---------|--------|-------------|
| **SUID** | `4000` | `u+s` | Execute file as the file owner |
| **SGID** | `2000` | `g+s` | Execute file as the file's group; new files inherit group |
| **Sticky Bit** | `1000` | `+t` | Only owner can delete files in directory |

```bash
# Set SUID (e.g., on passwd command)
sudo chmod u+s /usr/bin/someprogram
sudo chmod 4755 /usr/bin/someprogram

# Set SGID on directory (new files inherit group)
sudo chmod g+s /shared/directory
sudo chmod 2775 /shared/directory

# Set Sticky Bit (e.g., /tmp)
sudo chmod +t /shared/directory
sudo chmod 1777 /shared/directory
```

**Common examples:**
- `/usr/bin/passwd` has SUID → users can change their password (requires root access to shadow file)
- `/tmp` has Sticky Bit → anyone can write, but only delete their own files

---

## Sudo & Privilege Escalation

### The `sudoers` File

The `/etc/sudoers` file controls who can use `sudo` and what they can do.

> **⚠️ Never edit `/etc/sudoers` directly!** Always use `visudo` to prevent syntax errors that could lock you out.

```bash
# Edit sudoers file safely
sudo visudo
```

### Configuring Sudo Access

```bash
# Allow user to run ALL commands as ALL users
john ALL=(ALL:ALL) ALL

# Allow user to run commands without password
john ALL=(ALL) NOPASSWD: ALL

# Allow user to run specific commands only
john ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl restart apache2

# Allow group to use sudo
%sudo ALL=(ALL:ALL) ALL

# Allow user to run commands as specific user
john ALL=(root) /usr/bin/passwd
```

**Format breakdown:**
```
user    host=(runas)    command
john    ALL=(ALL:ALL)   ALL
```

| Field | Description |
|-------|-------------|
| `user` | Username or `%groupname` |
| `host` | Hostname (`ALL` for any) |
| `runas` | User and group to run as |
| `command` | Allowed commands (`ALL` for any) |

### Checking Sudo Privileges

```bash
# Check what sudo privileges you have
sudo -l

# Check if you have root privileges
sudo whoami
```

---

## Useful Commands Reference

### User Information

```bash
whoami              # Current username
id                  # Current user ID and group info
id username         # Info about specific user
who                 # Who is logged in
w                   # Who is logged in and what they're doing
last                # Last logged in users
lastlog             # Last login time for all users
finger username     # User information (if installed)
```

### Group Information

```bash
groups              # Groups current user belongs to
groups username     # Groups specific user belongs to
getent group groupname   # List members of a group
members groupname   # List group members (if installed)
```

### File Ownership & Permissions

```bash
ls -l               # List files with permissions
ls -la              # Include hidden files
ls -ld directory    # Info about directory itself
stat filename       # Detailed file info including permissions
namei -l /path      # Trace path and show permissions at each level
```

### Finding Command Locations

```bash
which command       # Find executable path
whereis command     # Find binary, source, and man pages
whatis command      # Short description of command
type command        # How command would be interpreted
```

### Process & Session Management

```bash
ps -u username      # Processes owned by user
killall -u username # Kill all processes by user
pkill -u username   # Send signal to user's processes
```

---

## Best Practices

### 1. **Principle of Least Privilege**
- Never give users more permissions than they need
- Use `sudo` instead of logging in as root
- Create separate accounts for different roles

### 2. **Strong Password Policies**
```bash
# Set minimum password length
sudo sed -i 's/PASS_MIN_LEN.*/PASS_MIN_LEN   12/' /etc/login.defs

# Set password expiration
sudo chage -M 90 -m 7 -W 14 username
```

### 3. **Use Groups for Permission Management**
- Create groups for projects/teams (e.g., `webdev`, `dbadmins`)
- Set SGID on shared directories so new files inherit the group
- Use `umask` to set default file permissions

```bash
# Set umask so new files are 640 and directories 750
umask 027
```

### 4. **Regular Audits**
```bash
# List users with UID 0 (should only be root)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Find users without passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Check for SUID/SGID files (potential security risks)
find / -perm /6000 -type f 2>/dev/null
```

### 5. **Secure Shared Directories**
```bash
# Create a shared project directory
sudo mkdir /projects/webapp
sudo chown root:webdev /projects/webapp
sudo chmod 2775 /projects/webapp    # rwxrwsr-x (SGID set)
```

---

## Further Reading

- [Linux Foundation - User and Group Management](https://www.linuxfoundation.org/)
- [Linode Docs - Linux Users and Groups](https://www.linode.com/docs/guides/linux-users-and-groups/)
- [Arch Wiki - Users and Groups](https://wiki.archlinux.org/title/Users_and_groups)
- [Ubuntu Server Guide - User Management](https://ubuntu.com/server/docs/user-management)
- [Red Hat - Managing Users and Groups](https://access.redhat.com/documentation/)

---

## Quick Cheat Sheet

| Task | Command |
|------|---------|
| Add user | `sudo useradd -m username` |
| Add user (interactive) | `sudo adduser username` |
| Set password | `sudo passwd username` |
| Delete user | `sudo userdel -r username` |
| Add group | `sudo groupadd groupname` |
| Delete group | `sudo groupdel groupname` |
| Add user to group | `sudo usermod -aG group user` |
| Remove user from group | `sudo gpasswd -d user group` |
| Change owner | `sudo chown user:group file` |
| Change permissions | `chmod 755 file` |
| Switch user | `su - username` |
| Run as root | `sudo command` |
| Check user info | `id username` |
| Check groups | `groups username` |

---

> **Contributions welcome!** If you find any errors or want to add more examples, feel free to open a Pull Request.

*Last updated: August 2026*
