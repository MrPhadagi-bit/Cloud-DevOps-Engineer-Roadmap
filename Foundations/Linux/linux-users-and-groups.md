# Linux Users & Groups

> A comprehensive guide to understanding and managing Linux users, groups, and permissions.
> Based on [Linux for Programmers #3 | Linux Users and Groups](https://youtu.be/b-9j2jiCOEA) by TechWithTim & Linode.

---

## Table of Contents

- [Introduction](#introduction)
- [Understanding Linux Users](#understanding-linux-users)
  - [Types of Users](#types-of-users)
  - [The `/etc/passwd` File](#the-etcpasswd-file)
  - [The `/etc/shadow` File](#the-etcshadow-file)
  - [Creating Users](#creating-users)
  - [Setting & Changing Passwords](#setting--changing-passwords)
  - [Switching Between Users](#switching-between-users)
  - [Deleting Users](#deleting-users)
- [Understanding Linux Groups](#understanding-linux-groups)
  - [Primary vs. Secondary Groups](#primary-vs-secondary-groups)
  - [The `/etc/group` File](#the-etcgroup-file)
  - [Creating Groups](#creating-groups)
  - [Assigning Users to Groups](#assigning-users-to-groups)
  - [Removing Users from Groups](#removing-users-from-groups)
  - [Deleting Groups](#deleting-groups)
- [File & Directory Permissions](#file--directory-permissions)
  - [Permission Types: Read, Write, Execute](#permission-types-read-write-execute)
  - [Viewing Permissions with `ls -l`](#viewing-permissions-with-ls--l)
  - [Changing Permissions with `chmod`](#changing-permissions-with-chmod)
    - [Symbolic Notation](#symbolic-notation)
    - [Octal (Numeric) Notation](#octal-numeric-notation)
  - [Changing Ownership with `chown`](#changing-ownership-with-chown)
  - [Changing Group with `chgrp`](#changing-group-with-chgrp)
  - [Special Permissions: SUID, SGID, Sticky Bit](#special-permissions-suid-sgid-sticky-bit)
- [Sudo & Privilege Escalation](#sudo--privilege-escalation)
  - [Understanding `sudo`](#understanding-sudo)
  - [The Sudoers File](#the-sudoers-file)
  - [Whitelisting Commands](#whitelisting-commands)
- [Practical Examples](#practical-examples)
- [Best Practices](#best-practices)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Introduction

Linux is a **multi-user operating system** designed from the ground up to allow multiple users to access the system simultaneously. Unlike single-user systems, Linux needs a robust mechanism to:

- **Isolate users** from each other
- **Protect files and processes** from unauthorized access
- **Enable collaboration** through shared resources
- **Grant administrative privileges** selectively

This is achieved through the **Users and Groups** system, combined with **file permissions**. Every file, directory, and process in Linux is associated with a specific user and group, and permissions determine who can read, modify, or execute them.

> **Key Concept:** In Linux, everything is a file — and every file has an **owner**, a **group**, and **permission bits** that control access.

---

## Understanding Linux Users

### Types of Users

Linux categorizes users into three main types:

| Type | Description | Examples |
|------|-------------|----------|
| **Root (Superuser)** | Has unlimited access to the entire system. Can read, modify, or delete any file; start/stop any process. | `root` (UID 0) |
| **System Users** | Created automatically during package installation. Used to run services/daemons. Cannot log in interactively. | `www-data`, `mysql`, `sshd`, `postfix` |
| **Regular Users** | Standard human users with limited privileges. Can only access their own files and shared resources. | `alice`, `bob`, `developer` |

> **UID (User ID):** Every user has a unique numeric identifier. Root is always `0`, system users typically range from `1-999`, and regular users start from `1000`.

---

### The `/etc/passwd` File

The `/etc/passwd` file is the central database of user accounts. Each line represents one user and contains seven colon-separated fields:

```bash
cat /etc/passwd
```

**Example entry:**
```
alice:x:1000:1000:Alice Smith:/home/alice:/bin/bash
```

| Field | Value | Description |
|-------|-------|-------------|
| 1 | `alice` | **Username** — the login name |
| 2 | `x` | **Password placeholder** — actual password is stored in `/etc/shadow` |
| 3 | `1000` | **UID** — unique user ID |
| 4 | `1000` | **GID** — primary group ID |
| 5 | `Alice Smith` | **GECOS** — full name / comment field |
| 6 | `/home/alice` | **Home directory** — user's personal directory |
| 7 | `/bin/bash` | **Default shell** — program run at login |

---

### The `/etc/shadow` File

This file stores **encrypted passwords** and password aging information. It is readable only by root for security reasons.

```bash
sudo cat /etc/shadow
```

**Example entry:**
```
alice:$6$rounds=5000$saltsalt$hashhashhash:19000:0:99999:7:::
```

| Field | Description |
|-------|-------------|
| 1 | Username |
| 2 | Encrypted password hash |
| 3 | Last password change (days since Jan 1, 1970) |
| 4 | Minimum days between password changes |
| 5 | Maximum days password is valid |
| 6 | Warning days before expiration |
| 7 | Inactive days after expiration |
| 8 | Account expiration date |

---

### Creating Users

There are two main commands for creating users:

#### `useradd` — Low-level, manual approach

```bash
# Create a basic user (no home directory by default)
sudo useradd bob

# Create user with home directory and default shell
sudo useradd -m -s /bin/bash bob

# Create user with specific home directory
sudo useradd -m -d /home/custom/bob -s /bin/bash bob

# Create user with expiration date
sudo useradd -e 2025-12-31 bob

# Create system user (no login, no home)
sudo useradd -r -s /usr/sbin/nologin serviceaccount
```

**Common `useradd` options:**

| Option | Description |
|--------|-------------|
| `-m` | Create home directory |
| `-d <dir>` | Specify custom home directory |
| `-s <shell>` | Set default shell |
| `-e <date>` | Set account expiration date (YYYY-MM-DD) |
| `-f <days>` | Set inactive days before account is disabled |
| `-r` | Create a system account |
| `-G <groups>` | Add to secondary groups |

#### `adduser` — High-level, interactive approach (Debian/Ubuntu)

```bash
# Interactive wizard — asks for password, full name, etc.
sudo adduser bob
```

**Output:**
```
Adding user 'bob' ...
Adding new group 'bob' (1001) ...
Adding new user 'bob' (1001) with group 'bob' ...
Creating home directory '/home/bob' ...
Copying files from '/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for bob
Enter the new value, or press ENTER for the default
    Full Name []: Bob Johnson
    Room Number []: 101
    Work Phone []: 555-0100
    Home Phone []:
    Other []:
Is the information correct? [Y/n] Y
```

> **Recommendation:** Use `adduser` for interactive creation on Debian/Ubuntu systems. Use `useradd` for scripts and automation.

---

### Setting & Changing Passwords

```bash
# As root: set password for another user
sudo passwd bob

# As a regular user: change your own password
passwd

# Force user to change password on next login
sudo passwd -e bob

# Lock a user account
sudo passwd -l bob

# Unlock a user account
sudo passwd -u bob
```

---

### Switching Between Users

```bash
# Switch to root user (requires root password)
su -

# Switch to another user (requires that user's password)
su - bob

# Execute a single command as root (requires sudo privileges)
sudo apt update

# Switch to another user and run a command
sudo -u bob whoami
```

> **The `-` flag:** `su -` (or `su -l`) simulates a full login, loading the target user's environment variables and landing in their home directory. Without it, you keep your current environment.

---

### Deleting Users

```bash
# Delete user but keep home directory and files
sudo userdel bob

# Delete user AND remove home directory and mail spool
sudo userdel -r bob

# Also remove the user's primary group if empty
sudo userdel -r bob
```

> **Warning:** `userdel` without `-r` leaves orphaned files in `/home`. Always verify what you want to keep before using `-r`.

---

## Understanding Linux Groups

Groups are a way to **organize users** and manage permissions collectively. Instead of assigning permissions to individual users, you assign them to a group — and any member of that group inherits those permissions.

### Primary vs. Secondary Groups

| Aspect | Primary Group | Secondary Group(s) |
|--------|---------------|-------------------|
| **Count** | Exactly one per user | Zero or more (max 15) |
| **Purpose** | Default group for new files | Additional access rights |
| **Stored in** | `/etc/passwd` (4th field) | `/etc/group` |
| **File creation** | New files belong to this group | Does not affect file ownership |
| **Change with** | `usermod -g` | `usermod -a -G` |

**View your groups:**
```bash
# Show all groups for current user
groups

# Show all groups for a specific user
groups alice

# Detailed ID info
id alice
```

**Example `id` output:**
```
uid=1000(alice) gid=1000(alice) groups=1000(alice),4(adm),24(cdrom),27(sudo),30(dip)
```

- `uid=1000(alice)` — User ID and name
- `gid=1000(alice)` — Primary group
- `groups=...` — All groups the user belongs to

---

### The `/etc/group` File

This file defines all groups on the system:

```bash
cat /etc/group
```

**Example entry:**
```
developers:x:1005:alice,bob,charlie
```

| Field | Value | Description |
|-------|-------|-------------|
| 1 | `developers` | Group name |
| 2 | `x` | Password placeholder (rarely used) |
| 3 | `1005` | GID — unique group ID |
| 4 | `alice,bob,charlie` | Member list (comma-separated usernames) |

---

### Creating Groups

```bash
# Create a new group
sudo groupadd developers

# Create a group with specific GID
sudo groupadd -g 2000 developers

# Create a system group
sudo groupadd -r sysgroup
```

---

### Assigning Users to Groups

```bash
# Add user to a secondary group (PRESERVES existing groups!)
sudo usermod -a -G developers alice

# Add user to multiple secondary groups at once
sudo usermod -a -G developers,adm,sudo alice

# Change user's PRIMARY group
sudo usermod -g developers alice

# Create user with specific primary and secondary groups
sudo useradd -g developers -G sudo,adm alice
```

> **CRITICAL:** Always use `-a` (append) with `-G`. Without `-a`, you **overwrite** all existing secondary groups, potentially locking yourself out of `sudo`!

**Apply group changes without logging out:**
```bash
# Start a new shell with updated group membership
newgrp developers

# Or log out and back in for changes to fully take effect
```

---

### Removing Users from Groups

```bash
# Remove user from a specific secondary group
sudo gpasswd -d alice developers

# Alternative: set the complete list of secondary groups (overwrites!)
sudo usermod -G sudo,adm alice   # developers is removed
```

---

### Deleting Groups

```bash
# Remove a group
sudo groupdel developers

# Remove a group and reassign files to another group
sudo groupdel -f developers
```

> **Note:** You cannot delete a group that is a user's primary group. Change the user's primary group first.

---

## File & Directory Permissions

### Permission Types: Read, Write, Execute

Linux uses three basic permission types, applied to three categories of users:

| Permission | Symbol | Value | On Files | On Directories |
|------------|--------|-------|----------|----------------|
| **Read** | `r` | 4 | View file contents | List directory contents (`ls`) |
| **Write** | `w` | 2 | Modify file contents | Create/delete/rename files |
| **Execute** | `x` | 1 | Run file as program | Enter directory (`cd`) |

**Three permission categories:**

| Category | Description |
|----------|-------------|
| **Owner (u)** | The user who owns the file |
| **Group (g)** | Members of the file's group |
| **Others (o)** | Everyone else on the system |

---

### Viewing Permissions with `ls -l`

```bash
ls -l /etc/passwd
ls -la ~/documents
```

**Example output:**
```
-rw-r--r--  1 alice developers  4096 Jan 15 09:30 report.txt
drwxr-x---  2 alice developers  4096 Jan 14 16:20 project/
```

**Permission string breakdown (`-rw-r--r--`):**

```
-  rw-  r--  r--
|   |    |    |
|   |    |    └── Others: r-- (read only)
|   |    └─────── Group:  r-- (read only)
|   └──────────── Owner:   rw- (read + write)
└──────────────── Type:    - (regular file)
```

**File type indicators:**

| Symbol | Type |
|--------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `s` | Socket |
| `p` | Named pipe |
| `c` | Character device |
| `b` | Block device |

---

### Changing Permissions with `chmod`

#### Symbolic Notation

Use letters to add (`+`), remove (`-`), or set exactly (`=`) permissions:

```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for group
chmod g-w file.txt

# Add read permission for others
chmod o+r file.txt

# Set exact permissions: owner=rwx, group=rx, others=r
chmod u=rwx,g=rx,o=r file.txt

# Add read for all (user, group, others)
chmod a+r file.txt

# Remove all permissions for others
chmod o-rwx file.txt

# Multiple changes at once
chmod u+x,g+w,o-r file.txt
```

**Symbolic reference table:**

| Who | Operator | Permission |
|-----|----------|------------|
| `u` = user (owner) | `+` = add | `r` = read |
| `g` = group | `-` = remove | `w` = write |
| `o` = others | `=` = set exactly | `x` = execute |
| `a` = all (ugo) | | `X` = execute (only if directory) |

#### Octal (Numeric) Notation

Each permission has a numeric value. Sum them up for each category:

| Permission | Value |
|------------|-------|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |
| None (`-`) | 0 |

**Common octal patterns:**

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| `777` | `rwxrwxrwx` | Full access for everyone |
| `755` | `rwxr-xr-x` | Owner full, others read+execute |
| `750` | `rwxr-x---` | Owner full, group read+execute, others nothing |
| `700` | `rwx------` | Owner only |
| `644` | `rw-r--r--` | Owner read+write, others read only |
| `640` | `rw-r-----` | Owner rw, group r, others nothing |
| `600` | `rw-------` | Owner only (private files) |
| `400` | `r--------` | Owner read only (immutable configs) |

```bash
# Make script executable
chmod 755 script.sh

# Private file — only owner can read/write
chmod 600 ~/.ssh/id_rsa

# Shared directory — owner and group can do anything, others nothing
chmod 770 /shared/project

# Web directory — owner full, group read+execute, others read+execute
chmod 755 /var/www/html
```

---

### Changing Ownership with `chown`

```bash
# Change owner
sudo chown alice file.txt

# Change owner and group
sudo chown alice:developers file.txt

# Change only group (using chown)
sudo chown :developers file.txt

# Recursive change for directories
sudo chown -R alice:developers /shared/project

# Reference another file's ownership
sudo chown --reference=template.txt newfile.txt
```

---

### Changing Group with `chgrp`

```bash
# Change group ownership
sudo chgrp developers file.txt

# Recursive
sudo chgrp -R developers /shared/project
```

---

### Special Permissions: SUID, SGID, Sticky Bit

Beyond the basic 9 permission bits, Linux has three special permissions:

| Special Bit | Symbol | Octal | On Files | On Directories |
|-------------|--------|-------|----------|----------------|
| **SUID** | `s` (owner) | 4000 | Execute with owner's privileges | — |
| **SGID** | `s` (group) | 2000 | Execute with group's privileges | New files inherit directory's group |
| **Sticky Bit** | `t` (others) | 1000 | — | Only owner can delete/rename files |

#### SUID (Set User ID)

When set on an executable, the file runs with the **owner's privileges**, not the runner's.

```bash
# Set SUID
chmod u+s /usr/bin/someprogram
chmod 4755 /usr/bin/someprogram

# Real-world example: passwd command
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
```

> The `s` in the owner's execute position indicates SUID is set. Regular users can run `passwd` to change their password because it executes with root privileges.

#### SGID (Set Group ID)

**On files:** Execute with the **file's group** privileges.

**On directories:** New files created in the directory inherit the **directory's group** instead of the user's primary group.

```bash
# Set SGID on a shared directory
chmod g+s /shared/project
chmod 2775 /shared/project

# Result: any file created here gets the directory's group
ls -ld /shared/project
# drwxrwsr-x 2 alice developers ... /shared/project
```

> The `s` in the group's execute position indicates SGID is set.

#### Sticky Bit

On a shared directory with write access for multiple users, the sticky bit ensures **only the file owner (or root) can delete or rename files** — even if others have write permission on the directory.

```bash
# Set sticky bit
chmod +t /tmp
chmod 1777 /tmp

ls -ld /tmp
# drwxrwxrwt 10 root root ... /tmp
```

> The `t` in the others' execute position indicates the sticky bit is set. `/tmp` is the classic example — everyone can write, but users can only delete their own files.

**Setting special permissions with octal:**

```bash
# SUID + regular permissions
chmod 4755 file      # rwsr-xr-x

# SGID + regular permissions
chmod 2755 file      # rwxr-sr-x

# Sticky + regular permissions
chmod 1755 dir       # rwxr-xr-t

# All three: SUID + SGID + Sticky
chmod 7755 file      # rwsrwsrwt
```

---

## Sudo & Privilege Escalation

### Understanding `sudo`

`sudo` (Superuser DO) allows authorized users to execute commands with **elevated privileges** (usually as root) without logging in as root. This provides:

- **Audit trail** — who ran what command and when
- **Granular control** — limit which commands users can run
- **Security** — no need to share the root password

```bash
# Run a command as root
sudo apt update

# Run a command as another user
sudo -u bob whoami

# Open a root shell
sudo -i

# Edit a file as root
sudo nano /etc/hosts
```

---

### The Sudoers File

The sudoers file (`/etc/sudoers`) defines who can use `sudo` and what they can do. **Never edit it directly with a text editor!** Always use `visudo`, which validates the syntax before saving.

```bash
# Edit sudoers file
sudo visudo
```

**Basic syntax:**
```
# user    host=(runas)    commands
alice     ALL=(ALL:ALL)   ALL
```

| Field | Meaning |
|-------|---------|
| `alice` | The user being granted privileges |
| `ALL` | Applies to all hosts |
| `(ALL:ALL)` | Can run as all users and all groups |
| `ALL` | Can run all commands |

**Grant sudo to a group:**
```
# % indicates a group
%developers ALL=(ALL:ALL) ALL
```

**Common sudoers entries:**

```bash
# Full sudo access for a user
alice ALL=(ALL:ALL) ALL

# Full sudo without password prompt (NOT recommended)
alice ALL=(ALL:ALL) NOPASSWD: ALL

# Allow only specific commands
bob ALL=(root) /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx

# Allow members of 'sudo' group to run any command
%sudo ALL=(ALL:ALL) ALL
```

---

### Whitelisting Commands

Follow the **principle of least privilege** — grant only the permissions users actually need.

```bash
# In sudoers file (via visudo):
# User can only restart and check nginx
wwwadmin ALL=(root) /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx

# Group can only use apt, ls, and less
%deployers ALL=(root) /usr/bin/apt-get, /usr/bin/ls, /usr/bin/less
```

**Find the absolute path to a command:**
```bash
which systemctl
# /usr/bin/systemctl

whereis nginx
# nginx: /usr/sbin/nginx /usr/lib/nginx ...
```

---

## Practical Examples

### Example 1: Setting Up a Development Team

```bash
# 1. Create a group for the dev team
sudo groupadd devteam

# 2. Create users and add them to the group
sudo adduser alice
sudo adduser bob
sudo usermod -a -G devteam alice
sudo usermod -a -G devteam bob

# 3. Create a shared project directory
sudo mkdir -p /projects/webapp
sudo chown root:devteam /projects/webapp
sudo chmod 2775 /projects/webapp   # SGID + rwxrwxr-x

# 4. Verify
ls -ld /projects/webapp
# drwxrwsr-x 2 root devteam 4096 ... /projects/webapp

# 5. Test: alice creates a file, bob should be able to edit it
su - alice
cd /projects/webapp
touch shared-config.yml
# File is created with devteam group thanks to SGID
```

### Example 2: Securing SSH Keys

```bash
# Create .ssh directory
mkdir ~/.ssh
chmod 700 ~/.ssh          # Owner only

# Add private key
chmod 600 ~/.ssh/id_rsa   # Owner read+write only
chmod 644 ~/.ssh/id_rsa.pub  # Public key can be readable

# Set correct ownership (if needed)
chown $USER:$USER ~/.ssh -R
```

### Example 3: Web Server Setup

```bash
# Create web user (if not exists)
sudo useradd -r -s /usr/sbin/nologin www-data

# Create web root
sudo mkdir -p /var/www/mysite
sudo chown -R www-data:www-data /var/www/mysite
sudo chmod 755 /var/www/mysite

# Allow deploy user to update files
sudo usermod -a -G www-data deployuser
sudo chmod 775 /var/www/mysite
sudo chmod g+s /var/www/mysite   # SGID for consistent group
```

### Example 4: Shared `/tmp` Directory Behavior

```bash
# The sticky bit on /tmp means:
# - Anyone can create files
# - Anyone can write to files they own
# - NO ONE can delete files they don't own

ls -ld /tmp
# drwxrwxrwt  ...  /tmp

# Create test files as different users
su - alice -c "touch /tmp/alice_file"
su - bob -c "touch /tmp/bob_file"

# Alice tries to delete bob's file (will fail)
su - alice -c "rm /tmp/bob_file"
# rm: cannot remove '/tmp/bob_file': Operation not permitted
```

---

## Best Practices

1. **Principle of Least Privilege**
   - Give users the minimum permissions they need
   - Use `sudo` with whitelisted commands instead of full root access

2. **Strong, Unique Passwords**
   - Never share passwords between accounts
   - Use password policies (expiration, complexity)

3. **Use Groups for Collaboration**
   - Create groups per project or function
   - Use SGID on shared directories to maintain consistent group ownership
   - Set `umask 002` for shared projects (group-writable by default)

4. **Secure Sensitive Files**
   - Private keys: `600` (`rw-------`)
   - `.ssh` directory: `700` (`rwx------`)
   - Configuration with secrets: `640` (`rw-r-----`)

5. **Regular Auditing**
   ```bash
   # List all users
   cat /etc/passwd | cut -d: -f1

   # List users with sudo access
   grep -Po '^sudo.+:\K.*' /etc/group
   getent group sudo

   # Check for users with UID 0 (should only be root)
   awk -F: '$3 == 0 {print $1}' /etc/passwd
   ```

6. **Avoid Root Login**
   - Disable direct root SSH login
   - Use regular user + `sudo` for administrative tasks
   - Lock root password if using sudo exclusively: `sudo passwd -l root`

7. **Clean Up After Departures**
   - Remove or disable accounts promptly when users leave
   - Archive home directories before deletion if needed

---

## Quick Reference Cheat Sheet

### Users

| Task | Command |
|------|---------|
| Create user (basic) | `sudo useradd username` |
| Create user (with home) | `sudo useradd -m username` |
| Create user (interactive) | `sudo adduser username` |
| Set password | `sudo passwd username` |
| Delete user | `sudo userdel username` |
| Delete user + home | `sudo userdel -r username` |
| Modify user | `sudo usermod [options] username` |
| View user info | `id username` |
| List all users | `cat /etc/passwd` |
| Switch user | `su - username` |

### Groups

| Task | Command |
|------|---------|
| Create group | `sudo groupadd groupname` |
| Delete group | `sudo groupdel groupname` |
| Add user to group | `sudo usermod -a -G groupname username` |
| Remove user from group | `sudo gpasswd -d username groupname` |
| Change primary group | `sudo usermod -g groupname username` |
| View user's groups | `groups username` |
| List all groups | `cat /etc/group` |

### Permissions

| Task | Command |
|------|---------|
| View permissions | `ls -l file` / `ls -la dir` |
| Change permissions (symbolic) | `chmod u+rwx,g+rx,o-rwx file` |
| Change permissions (octal) | `chmod 755 file` |
| Change owner | `sudo chown user file` |
| Change owner & group | `sudo chown user:group file` |
| Change group | `sudo chgrp group file` |
| Set SUID | `chmod u+s file` / `chmod 4755 file` |
| Set SGID | `chmod g+s dir` / `chmod 2775 dir` |
| Set Sticky Bit | `chmod +t dir` / `chmod 1777 dir` |

### Sudo

| Task | Command |
|------|---------|
| Run command as root | `sudo command` |
| Edit sudoers safely | `sudo visudo` |
| Run as specific user | `sudo -u username command` |
| Root shell | `sudo -i` |

---

## Additional Resources

- [Linode Docs: Linux Users and Groups](https://www.linode.com/docs/guides/linux-users-and-groups/)
- [Linux `man` pages](https://man7.org/linux/man-pages/): `man useradd`, `man usermod`, `man chmod`, `man chown`
- [Debian System Administrator's Manual](https://www.debian.org/doc/manuals/debian-reference/)

---

> **License:** This guide is provided as-is for educational purposes. Feel free to use, modify, and distribute.
> 
> **Contributing:** Found an error or want to improve this guide? PRs are welcome!
